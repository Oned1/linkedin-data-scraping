# How to Webscrape Linkedin with Python?
I was looking for the answer of this question when i started doing a little side project with my friend. Apperiently there is no good open source linkedin web scraper in the market.<br><br>
The ones i saw was either paid or poorly made. Like the ones that only scrapes titles of the listed jobs and doesn't click on it to scrape more complex details.<br><br>
It is important to point out what exactly i wanted to web scrape. So let me wrap it out with a quick summary of what i intended to do.<br><br>
# Plan
**1. Deciding The Source of Data -** I wanted to analyze and list various job advertisement data, there are many job posting websites but the cleanest data i could find was in Linkedin. That is why i made my choice in that direction.<br><br>
**2. Scraping/Collecting Job Data -** There were many parameters that i wanted to collect such as, title of the job, location of the job, name of the company, date of the post. These were the easy part, for tougher part we have job description and working type of the jobs. These two requires us to click the job post and wait for it to load, but that is not the issue. The issue is "working type" of the jobs is kind of unpredictable. It changes depending on the firm that posted the job. So it doesn't have a constant location or an xpath.<br><br>
**3. Importing The Data To Our Database -** For this project i used MySQL, you can also save your collected data in a CSV with little changes in the code. I used MySQL for better visualization purposes and to give you a better start for some project types.<br><br>

# Required Components
**Python -** You can easily install Python from it's official website. It is free and it doesn't require a license. To be able to write your code better at Python i would suggest you to use a code editor such as Visual Studio Code.<br><br>
**Selenium -** It is a widely used open source webscraping framework for Python and many other coding languages. To install it in your Python you should use the command below in your cmd.exe (Windows Command Processor). 
```python
pip install selenium
```
If you use another operating system than windows than steps might be a little different and you also need to install a Selenium webdriver to be able to run Selenium code. But i won't go deep in Selenium installation process. If you want to get more info about Selenium Installation and Selenium in general than visit this site <br><br>
*https://selenium-python.readthedocs.io/installation.html*
# Reuqired Libraries and MySQL connection
```python
#Reqired libraries for the project.
from asyncio.windows_events import NULL
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.action_chains import ActionChains
import mysql.connector
import time

#Connection to your local database.
db = mysql.connector.connect(
    host = "localhost",
    user = "root",
    password = "",
    database = "selenium_test"
)
```
# Searching For The Given Job In Linkedin
In the code block below we are creating our class and defining our Selenium Driver Location. For some people the driver might work without a given path but i am going to use it in this way.<br><br>
In the function of "search()" we are going to build our way to Linkedin's job search page and put our given parameters in the inputs of that page.<br><br>
We are giving a time.sleep for one second because we want our page to be fully loaded before putting our inputs in the input boxes.<br><br>
```python
mycursor = db.cursor()

driver = webdriver.Chrome("C:\Drivers\chromedriver")
class Linkedin:
    def __init__(self, job, location):
        self.job = job
        self.location = location
    def search(self):
        driver.get("https://www.linkedin.com/jobs")
        driver.maximize_window()
        time.sleep(1)
        
        name_Input = driver.find_element("name", "keywords")
        name_Input.send_keys(self.job)
    
        location_Input = driver.find_element("name", "location")
        location_Input.clear()
        location_Input.send_keys(self.location)
        location_Input.send_keys(Keys.ENTER)
```
# Pulling The Job Data From Linkedin
At first we are pulling the page to the bottom with one second pauses for our browser to load all job posts in the page. That is because Linekdin is using a scroll-to-load system with the help of Javascript.<br><br>
After loading all job posts, we are pulling job datas one by one with the help of an index system so we won't lose the order of data. <br><br>
After that and some little error handling, we are inserting our freshly scraped data to our MySQL database.
```python
 def jobList(self):
        time.sleep(3)

        for x in range (10):
            driver.execute_script("window.scrollTo(0,document.body.scrollHeight)")
            time.sleep(1)
        
        job_Titles = driver.find_elements(By.CSS_SELECTOR, ".jobs-search__results-list h3")
        job_Locations = driver.find_elements(By.CSS_SELECTOR, ".job-search-card__location")
        job_CompanyNames = driver.find_elements(By.CSS_SELECTOR, ".jobs-search__results-list h4")
        job_PostDates = []
            
        for x in range (len(job_Titles)):
            print(f"[{x+1}]:")
            print(job_Titles[x].text)
            print(job_Locations[x].text)
            print(job_CompanyNames[x].text)
            y = x + 1
            try:
                a = driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{y}]/div/div[2]/div/time').text
                job_PostDates.append(a)
                print(driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{y}]/div/div[2]/div/time').text)
            except:       
                b = (driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{y}]/a/div[2]/div/time').text)
                job_PostDates.append(b)
                print(driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{y}]/a/div[2]/div/time').text)         

            print("\n")
            mycursor.execute(f"INSERT INTO jobs (title, location, company_name, post_date) VALUES ('{job_Titles[x].text}', '{job_Locations[x].text}', '{job_CompanyNames[x].text}', '{job_PostDates[x]}');")
            db.commit() 
```
# Clicking To Job Posts
Now that easy part is over, we can build our clicking system. In the code block below, we are simply clicking to the job posts and pulling their description data and working type data (Internship / Full Time / Part Time etc.). It is important to point that there is an issue in Linkedin. Time to time you will see a blank white screen when your webscraper clicks on some job posts. When you get that white screen you won't be able to pull your data.<br><br>
You might think that giving a "time.sleep()" will do the work and make our webscraper wait until the data inside job loads. But that is not the situation. Sometimes when that white screen appears it won't pass by waiting. You will need to go to the job post you clicked before this one and wait a little inside that, than try entering your intended job post again and it will probably work. If it doesn't than your bot will repat this process until the data loads and that will simply fix the empty data problem.<br><br>
```python
        job_Type = NULL
        job_Description = NULL
        click_Count = 1
        for click in range (150):
            try:
                driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{click_Count}]/div/a').click()
            except:
                try:  
                    driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{click_Count}]/a').click()
                except:
                    break
              
            j = 0
            while(True):
                time.sleep(0.5)
                try:
                    job_Type = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/ul/li[2]/span').text
                except:   

                    try:
                        job_Type = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/ul/li/span').text                                   
                    except:
                        job_Type = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section/div/ul/li/span').text
                
                if(job_Type != ""):
                    try:
                        job_Description = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/div/section/div').text
                    except:
                        job_Description = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/div[2]/section/div').text
                    break
                j = j + 1
                if(j > 5):
                    try:
                        driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{click_Count-1}]/div/a').click()
                        time.sleep(1)
                        driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{click_Count}]/div/a').click()
                        
                        time.sleep(0.5)
                        try:
                            job_Type = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/ul/li[2]/span').text
                        except:   

                            try:
                                job_Type = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/ul/li/span').text                                   
                            except:
                                job_Type = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section/div/ul/li/span').text
                        
                        if(job_Type != ""):
                            try:
                                job_Description = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/div/section/div').text
                                if(job_Type != ""):
                                    break
                            except:
                                job_Description = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/div[2]/section/div').text
                                break
                        

                    except:
                        driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{click_Count-1}]/a').click() 
                        time.sleep(1)
                        driver.find_element(By.XPATH, f'//*[@id="main-content"]/section[2]/ul/li[{click_Count}]/div/a').click()
                        
                        time.sleep(0.5)
                        try:
                            job_Type = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/ul/li[2]/span').text
                        except:   

                            try:
                                job_Type = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/ul/li/span').text                                   
                            except:
                                job_Type = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section/div/ul/li/span').text
                        
                        if(job_Type != ""):
                            try:
                                job_Description = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/div/section/div').text
                            except:
                                job_Description = driver.find_element(By.XPATH, '/html/body/div[1]/div/section/div[2]/div/section[1]/div/div[2]/section/div').text
                            break
                        
```
To explain the code better, what we are doing in this code block is trying to click to the job posts and pull their description and working type data. You can exaggerate this and pull other data types that is stated inside of the job post.<br><br>
Now, as i stated at the start of this article, our job type data (Internship / Full Time etc.) doesn't have a constant place or path. It changes depending on how firms post their job advertisements. But no need to worry, even tho it changes its place, it only have 3–4 alternative place and it will be in one of them. So we are checking all those places until we find the data we look for.<br><br>
# Importing Other Datas We Pulled To Our Database
When we first imported our datas to our database we left our description and job type empty, now we will update those two with the datas we currently pulled by clicking to the job post.<br><br>
We are going to replace our quote(") symbols with single quotes(') so it won't cause complications in out SQL query.<br><br>
After that we can call our functions at the end and use our web scraper by giving it parameters. And we are starting a timer and calculating how long it last to run our web scraper. This part is optional but you can optimize your code using this data or use it for other purposes, it is up to you.
```python
            job_Description = job_Description.replace('"', "'")
            mycursor.execute(f'UPDATE jobs SET description = "{job_Description}" WHERE id = {click_Count};')
            db.commit() 
            mycursor.execute(f"UPDATE jobs SET job_type = '{job_Type}' WHERE id = {click_Count};")
            db.commit()         
            print(f"[{click_Count}]: {job_Type}")
            print(f"[{click_Count}]: {job_Description[:300]}...")
            #time.sleep(0.5)
 
            click_Count += 1

#THIS PART IS OUTSIDE OF THE CLASS
start = time.time()

linkedin = Linkedin("C++", "istanbul") #You should put your own parameters here.
linkedin.search()
linkedin.jobList() 

end = time.time()    
print(f"\nExecution time: {end - start}")
```
***Thanks for reading, this was one of my learning projects. I am still trying to improve myself and i am open to any advice.***<br><br>
My contacts:<br>
Discord: Deno#7007<br>
Email: denizbarankara3@gmail.com<br>
Linkedin: https://www.linkedin.com/in/deniz-kara-219996252/<br>
