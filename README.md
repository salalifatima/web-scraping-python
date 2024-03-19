import pyodbc
import requests
from bs4 import BeautifulSoup

server = 'localhost'
database = 'master'
username = 'SA'
password = 'MyStrongPass123'
driver = '{SQL Server}'

connection_string = f'DRIVER={driver};SERVER={server};DATABASE={database};UID={username};PWD={password}'

connection = pyodbc.connect(connection_string)
cursor = connection.cursor()


urls = ["https://bina.az/alqi-satqi?page={}".format(i) for i in range(1, 2794)]

for url in urls:
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')
        ev_links = soup.find_all('div', class_='items-i')
        if ev_links:
            for ev_link in ev_links:
                link = "https://bina.az{}".format(ev_link.find('a')['href'])
                linked_page_response = requests.get(link, headers=headers)
                if linked_page_response.status_code == 200:
                    linked_soup = BeautifulSoup(linked_page_response.text, 'html.parser')
                    kateqoriyalar = linked_soup.find_all('div', class_='product-properties__i')
                    if kateqoriyalar:
                        for kateqoriya in kateqoriyalar:
                            kateqoriya_in_name = kateqoriya.find('label', class_='product-properties__i-name')
                            kateqoriya_in_val = kateqoriya.find('span', class_='product-properties__i-value')
                            if kateqoriya_in_name and kateqoriya_in_val:
                                cursor.execute("INSERT INTO YourTableName (category) VALUES (?)",
                                               (kateqoriya_in_name.text, kateqoriya_in_val.text))
                    
                    sekiller = linked_soup.find_all('div', class_='product-photos__slider-nav-i js-open-gallery')
                    if sekiller:
                        for sekil in sekiller:
                            img_url = sekil.find('div')['style']
                            cursor.execute("INSERT INTO YourTableName (img) VALUES (?)",
                                           (img_url))
                    
                    maps = linked_soup.find_all('div', class_='product-map__left__address')
                    if maps:
                        for map in maps:
                             cursor.execute("INSERT INTO YourTableName (map) VALUES (?)",
                                           (map.text))                            
                    infos = linked_soup.find_all('div', class_='product-description__content')
                    if infos:
                        for info in infos:
                            cursor.execute("INSERT INTO YourTableName (info) VALUES (?)",
                                           (info.text))
                connection.commit()            
                nomreler = ["{}/phones".format(link)]
                for nomre in nomreler:
                    nomre_response=requests.get(nomre,headers=headers)  
                    if nomre_response.status_code==200:
                        nomre_soup=BeautifulSoup(nomre_response.text,'html.parser')
                        if nomre_soup:
                            for number in nomre_soup:
                                cursor.execute("INSERT INTO YourTableName (nomre) VALUES (?)",
                                           (nomre.text))
                connection.commit()
                
connection.close()               
                

     
