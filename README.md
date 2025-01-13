# Weather Dashboard

This project is a Weather Dashboard that fetches and displays weather data for a specified city. Below are the steps taken to create this project, including explanations of the code used.

## Project Setup

1. **Create Project Directory**:
   ```sh
   mkdir weather-dashboard-demo
   cd weather-dashboard-demo
   ```

2. **Initialize Git Repository**:
   ```sh
   git init
   ```

3. **Create Project Files and Folders**:
   ```sh
   mkdir data src tests
   touch .env .gitignore requirements.txt README.md src/__init__.py src/weather_dashboard.py
   ```

4. **Add Dependencies**:
   Add the following dependencies to `requirements.txt`:
   ```txt
   boto3==1.26.137
   python-dotenv==1.0.0
   requests==2.28.2
   ```

5. **Configure .gitignore**:
   Add the following lines to `.gitignore` to exclude environment files and cache:
   ```txt
   .env
   __pycache__/
   *.zip
   ```

## Code Explanation

### `src/weather_dashboard.py`

This file contains the main code for fetching and displaying weather data.

1. **Import Libraries**:
   ```py
   import os
   import json
   import boto3
   import requests
   from datetime import datetime
   from dotenv import load_dotenv
   ```

2. **Load Environment Variables**:
   ```py
   load_dotenv()
   ```

3. **Define `WeatherDashboard` Class**:
   ```py
   class WeatherDashboard:
       def __init__(self):
           self.api_key = os.getenv('OPENWEATHER_API_KEY')
           self.bucket_name = os.getenv('AWS_BUCKET_NAME')
           self.s3_client = boto3.client('s3')
   ```

4. **Create S3 Bucket if Not Exists**:
   ```py
   def create_bucket_if_not_exists(self):
       try:
           self.s3_client.head_bucket(Bucket=self.bucket_name)
           print(f"Bucket {self.bucket_name} exists")
       except:
           print(f"Creating bucket {self.bucket_name}")
       try:
           self.s3_client.create_bucket(Bucket=self.bucket_name)
           print(f"Successfully created bucket {self.bucket_name}")
       except Exception as e:
           print(f"Error creating bucket: {e}")
   ```

5. **Fetch Weather Data**:
   ```py
   def fetch_weather(self, city):
       base_url = "http://api.openweathermap.org/data/2.5/weather"
       params = {
           "q": city,
           "appid": self.api_key,
           "units": "imperial"
       }
       try:
           response = requests.get(base_url, params=params)
           response.raise_for_status()
           return response.json()
       except requests.exceptions.RequestException as e:
           print(f"Error fetching weather data: {e}")
           return None
   ```

6. **Save Weather Data to S3**:
   ```py
   def save_to_s3(self, weather_data, city):
       if not weather_data:
           return False
       timestamp = datetime.now().strftime('%Y%m%d-%H%M%S')
       file_name = f"weather-data/{city}-{timestamp}.json"
       try:
           weather_data['timestamp'] = timestamp
           self.s3_client.put_object(
               Bucket=self.bucket_name,
               Key=file_name,
               Body=json.dumps(weather_data),
               ContentType='application/json'
           )
           print(f"Successfully saved data for {city} to S3")
           return True
       except Exception as e:
           print(f"Error saving to S3: {e}")
           return False
   ```

7. **Main Function**:
   ```py
   def main():
       dashboard = WeatherDashboard()
       dashboard.create_bucket_if_not_exists()
       cities = ["Philadelphia", "Seattle", "New York"]
       for city in cities:
           print(f"\nFetching weather for {city}...")
           weather_data = dashboard.fetch_weather(city)
           if weather_data:
               temp = weather_data['main']['temp']
               feels_like = weather_data['main']['feels_like']
               humidity = weather_data['main']['humidity']
               description = weather_data['weather'][0]['description']
               print(f"Temperature: {temp}°F")
               print(f"Feels like: {feels_like}°F")
               print(f"Humidity: {humidity}%")
               print(f"Conditions: {description}")
               success = dashboard.save_to_s3(weather_data, city)
               if success:
                   print(f"Weather data for {city} saved to S3!")
           else:
               print(f"Failed to fetch weather data for {city}")

   if __name__ == "__main__":
       main()
   ```

## Running the Project

1. **Install Dependencies**:
   ```sh
   pip install -r requirements.txt
   ```

2. **Run the Script**:
   ```sh
   python src/weather_dashboard.py
   ```

This will fetch weather data for the specified cities and save the data to the configured S3 bucket.