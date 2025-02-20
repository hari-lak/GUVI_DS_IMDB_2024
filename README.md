# GUVI_DS_IMDB_2024
pip install selenium
pip install pandas
import time
import pandas as pd
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys

driver = webdriver.Chrome()

driver.get("https://www.imdb.com/search/title/?title_type=feature&release_date=2024-01-01,2024-12-31&genres=adventure")

time.sleep(1)

def click_load_more():
    try:
        load_more_button = driver.find_element(By.XPATH,'//*[@id="__next"]/main/div[2]/div[3]/section/section/div/section/section/div[2]/div/section/div[2]/div[2]/div[2]/div/span/button')
        ActionChains(driver).move_to_element(load_more_button).perform()
        load_more_button.click()
        time.sleep(5)
        return True
    except Exception as e:
        print ("Error Clicking Load More Button :",e)
        return False

while click_load_more():
    print("Clicked Load More Button")

print("No more content to load")

time.sleep(1)

titles = []
genres = []
ratings = []
votings = []
durations = []

movie_items = driver.find_elements(By.XPATH,'//*[@id="__next"]/main/div[2]/div[3]/section/section/div/section/section/div[2]/div/section/div[2]/div[2]/ul/li')

for movie_item in movie_items:
    try:
        title = movie_item.find_element(By.XPATH,'./div/div/div/div[1]/div[2]/div[1]/a/h3').text
        genre = movie_item.find_element(By.XPATH,'//*  
        [@id="__next"]/main/div[2]/div[3]/section/section/div/section/section/div[2]/div/section/div[1]/div/div/div[2]/button[3]').text
        rating = movie_item.find_element(By.XPATH,'./div/div/div/div[1]/div[2]/span/div/span/span[1]').text
        voting = movie_item.find_element(By.XPATH,'./div/div/div/div[1]/div[2]/span/div/span/span[2]').text
        duration = movie_item.find_element(By.XPATH,'./div/div/div/div[1]/div[2]/div[2]/span[2]').text

        titles.append(title)
        genres.append(genre)
        ratings.append(rating)
        votings.append(voting)
        durations.append(duration)

    except Exception as e:
        print("Error Extractig data for the movie :")
        continue

dfadven = pd.DataFrame({
    'Title' : titles,
    'Genre' : genres,
    'Rating' : ratings,
    'Voting' : votings,
    'Duration' : durations
})

driver.quit()
dfadven
dfadven['Title'] = dfadven['Title'].apply(lambda x: ''.join([char for char in x if char not in '0123456789.']))
dfadven ['Voting']=dfadven ['Voting'].apply(lambda x: x.replace('(',''))
dfadven ['Voting']=dfadven ['Voting'].apply(lambda x: x.replace(')',''))
dfadven ['Voting']=dfadven ['Voting'].apply(lambda x: int(float(x[:-1]) * 1000) if x[-1] == 'K' else int(x))
dfadven['Rating'] = dfadven['Rating'].apply(lambda x: float(x))
dfadven['Duration'] = dfadven['Duration'].apply(lambda time_str: 
    
    (int(time_str.split('h')[0].strip()) * 60 + int(time_str.split('h')[1].split('m')[0].strip())) 
    if isinstance(time_str, str) and 'h' in time_str and 'm' in time_str else 
    
    (int(time_str.replace('m', '').strip()) if isinstance(time_str, str) and 'm' in time_str else 
    
    (int(time_str.replace('h', '').strip()) * 60 if isinstance(time_str, str) and 'h' in time_str else     
   
    time_str)
    ))
    dfadven.to_csv('dataadven.csv')
    from pandas import Series, DataFrame
import numpy as np
import pandas as pd
import glob
glob.glob('data*.csv')

master_df = pd.concat([pd.read_csv(single_df) for single_df in glob.glob('data*.csv')])
master_df.to_csv('master_df.csv')
pip install streamlit
pip install matplotlib
import pandas as pd
import streamlit as st


def load_data():
    
    df = pd.read_csv('master_df.csv')
    return df


def process_data(df):
    df['Rating'] = pd.to_numeric(df['Rating'], errors='coerce')
    df['Voting'] = pd.to_numeric(df['Voting'], errors='coerce')
    
    df = df.dropna(subset=['Rating', 'Voting'])
    
    top_movies = df.sort_values(by=['Rating', 'Voting'], ascending=False).head(10)
    
    return top_movies

def main():
    st.title("Top 10 Movies by Rating and Voting Count")
    
    df = load_data()
    
    top_movies = process_data(df)
    
    st.write("Top 10 Movies:")
    st.dataframe(top_movies[['Title', 'Rating', 'Voting']])
if __name__ == "__main__":
    main()
                   
