# ICO Project Code

## Data Collection

In this stage, we collected data of white papers for ICOs in a bid to further exploratory data analysis.

```Python

from bs4 import BeautifulSoup
from urllib.request import urlopen
import requests
import re
import pandas as pd 

# collect URLs of white papers in PDF format
html = requests.get('https://icowatchlist.com/finished').text
soup = BeautifulSoup(html, 'lxml')

ico_links = soup.find_all('a',{'href':re.compile('ico/')})
ico_links=[l['href'] for l in ico_links]
ico_links=ico_links[::2]

pdf=[]
companyname=[]
for i in ico_links:
    html2 = requests.get('https://icowatchlist.com/'+i).text
    soup2 = BeautifulSoup(html2, 'lxml')
    pdf_link=soup2.find_all('a',{"href":re.compile('.*?\.pdf')})
    for link in pdf_link:
        pdf.append(link['href'])
        try:
            companyname.append(link['data-slug'])
            except:
                companyname.append('None')
                
# collect initial price for existed ICOs
price=[]
for i in companyname:
    html3 = requests.get('https://coinmarketcap.com/currencies/'+i+'/historical-data/?start=20130428&end=20181004').text
    soup3 = BeautifulSoup(html3,'lxml')
    try:
        tables = soup3.find_all('table')
        tab = tables[0]
        price_table =[]
        for tr in tab.find_all('tr'):
            for td in tr.find_all('td'):
                price_table.append(td.getText())
        price.append(price_table[-6])
    except:
        price.append(None)

d={'Names':pd.Series(companyname),'URL':pd.Series(pdf),'Prices':pd.Series(price)}
df=pd.DataFrame(d)
df.dropna().to_csv('ICO_Rong.csv', encoding='utf-8', index=False)

```

