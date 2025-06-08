# BANGLORE HOUSE PRICE PREDICTOR
A machine learning web app that predicts house prices in Bangalore based on user inputs like location, BHK, and square footage. Built with Flask, scikit-learn, and deployed using NGINX on AWS EC2.

## Tech Stack

| Layer       | Technologies & Tools                 |
|-------------|------------------------------------|
| Frontend    | HTML, CSS, JavaScript               |
| Backend     | Python Flask                       |
| Machine Learning Model | scikit-learn                 |
| Data Processing & Visualization | NumPy, Pandas, Matplotlib  |
| Development IDEs | Jupyter Notebook, VS Code, PyCharm |
| Deployment  | AWS EC2, NGINX    

## Data Processing & Model Building

**Data Cleaning:**
- Removed irrelevant columns (`area_type`, `availability`, `society`).
- Dropped rows with missing values.
- Standardized `Size` (BHK) column to integers.
- Cleaned `Sqft` values by converting ranges to averages and removing invalid entries.

**Feature Engineering:**
- Created `Price per sq.ft` feature from `Price` and `Total Sqft`.
- Cleaned `Location` by trimming spaces and grouping rare locations (<10 occurrences) into 'Others'.

**Outlier Removal:**
- Filtered out properties with unrealistic sqft per bedroom (<300 sqft).
- Removed outliers in price per sq.ft grouped by location to account for area-wise price variation.
- Excluded properties with an unusually high number of bathrooms compared to bedrooms.

**Modeling:**
- Applied one-hot encoding to categorical features.
- Used Linear Regression as the predictive model.
- Validated model performance using K-Fold Cross-Validation with ShuffleSplit.
- Tuned hyperparameters using GridSearchCV to find the best model.

##  Run Locally

 1. Clone the repository
```
git clone https://github.com/yourusername/bangalore-house-price-predictor.git
cd bangalore-house-price-predictor
```
2. Set up virtual environment
```
cd server
python3 -m venv venv
source venv/bin/activate
```
3. Install dependencies
```
pip install -r requirements.txt
```
4. Run the Flask server
```
python3 server.py
```
5. Open the frontend
Open client/app.html manually in your browser, or use Live Server extension in VSCode

## Deploy this app to cloud (AWS EC2)

1. Create EC2 instance using amazon console, also in security group add a rule to allow HTTP incoming traffic
2. Now connect to your instance using a command like this,
```
ssh -i "C:\Users\Nikil\.ssh\Banglore.pem" ubuntu@ec2-3-133-88-210.us-east-2.compute.amazonaws.com
```
3. nginx setup
   1. Install nginx on EC2 instance using these commands,
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   2. Above will install nginx as well as run it. Check status of nginx using
   ```
   sudo service nginx status
   ```
   3. Here are the commands to start/stop/restart nginx
   ```
   sudo service nginx start
   sudo service nginx stop
   sudo service nginx restart
   ```
   4. Now when you load cloud url in browser you will see a message saying "welcome to nginx" This means your nginx is setup and running.
4. Now you need to copy all your code to EC2 instance. You can do this either using git or copy files using winscp.
5. Once you connect to EC2 instance from winscp, you can now copy all code files into /home/ubuntu/ folder. The full path of your root folder is now: **/home/ubuntu/BHP**
6.  After copying code on EC2 server now we can point nginx to load our property website by default. For below steps,
    1. Create this file /etc/nginx/sites-available/bhp.conf. The file content looks like this,
    ```
    server {
	    listen 80;
            server_name bhp;
            root /home/ubuntu/BHP/client;
            index app.html;
            location /api/ {
                 rewrite ^/api(.*) $1 break;
                 proxy_pass http://127.0.0.1:5000;
            }
    }
    ```
    2. Create symlink for this file in /etc/nginx/sites-enabled by running this command,
    ```
    sudo ln -v -s /etc/nginx/sites-available/bhp.conf
    ```
    3. Remove symlink for default file in /etc/nginx/sites-enabled directory,
    ```
    sudo unlink default
    ```
    4. Restart nginx,
    ```
    sudo service nginx restart
    ```
7. Now install python packages and start flask server
    ```
    sudo apt-get install python3-pip
    sudo pip3 install -r /home/ubuntu/BHP/server/requirements.txt
    python3 /home/ubuntu/BHP/client/server.py
    ```
   Running last command above will prompt that server is running on port 5000.

8. Now just load your cloud url in browser (for me it was http://ec2-13-53-148-180.eu-north-1.compute.amazonaws.com/) and this will be fully functional website running in production cloud environment
