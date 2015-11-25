import requests,re,urllib2,random,sys
import pandas as pd
import bs4 as bs
from pandas import DataFrame as df
import numpy as np
from datetime import datetime,timedelta
import time as T

def extract_area(x): #extract 123 from 'area 123 square feet'
    x=x.replace(',','')
    recompile=re.compile(r'[0-9]+')
    result=re.findall(recompile,x)
    if len(result)<>0:
        return float(result[0])      
    else:
        return np.nan
        
def extract_price(x): #extract 1,000,000 from string 'asd 1,2000,000 as1234'
    recompile=re.compile(r'\d+((,\d+)+)?((.\d+))+?')
    result=re.search(recompile,x)
    if not result==None:
        return float(result.group().replace(',',''))
    else:
        return np.nan

#property type mapping in iproperty
iproperty_type={'AR':'All Residence','A':'Apartment/Flat','E':'Condominium/Service Aprt.',\
'T':'Terrace/Link House/Townhouse','B':'Semi-D/Bungalow'}

with open('/Users/Daniel/Desktop/user_agents.txt','r') as f:
    ua=f.readlines()

def iproperty(keyword='danau murni',sr='S',proptype='AR'):
    keyword=keyword.replace(' ','+') #search keyword
    sr='S' if sr not in ['S','R'] else sr
    #proptype value: AR=all residence; A=apartment/flat; E=condo/s.aprt; T=terrace/link/townhouse
    #cont proptype value: B=semi-D/bungalow
    proptype='AR' if proptype not in ['AR','A','E','T','B'] else proptype
    
    #first='http://www.iproperty.com.my/property/searchresult.aspx?t=S&gpt=AR&st=&'+ \
    #'ct=&k='
    #third='&pt=&mp=&xp=&mbr=&xbr=&mbu=&xbu=&lo=&wp=&wv=&'
    #fifth='wa=&ht=&au=&ft=&sby=&pg='
    found=False #initiate the search validation indicator, to validate if any result is found
    dic={'name':[],'loc':[],'type':[],'furnish':[],'built_up':[],'posted':[],'price':[],'agent':[],'contact':[]} 
    
    for pg_num in xrange(1,100):
        path='http://www.iproperty.com.my/property/searchresult.aspx?'+ \
        't={0}&gpt=AR&st=&ct=&k={1}&pt=&'.format(sr,keyword)+ \
        'mp=&xp=&mbr=&xbr=&mbu=&xbu=&lo=&wp=&wv=&wa=&ht=&au=&ft=&sby=&pg={0}'.format(str(pg_num))
        #path=first+keyword+third+fifth+str(pg_num)
    
        res=requests.get(path)
        verify=int(res.url.split('=')[-1])
        if verify<>pg_num:
            #to quit the loop if exceeded the last page
            break
        
        soup=bs.BeautifulSoup(res.text.'html.parser')
        search_list=soup.select('.search-listing') #get all the search result in a list
        try:
            if len(search_list)<>0: #check if return any search result
                found=True
                for i in xrange(len(search_list)): #loop through every search result of a single page
                    propertyinfo=search_list[i].select('.article-left')[0].getText()
                    propertyinfo=propertyinfo.splitlines() #first item is empty
                    propertyinfo=propertyinfo[1:] #drop the first empty item
                    
                    colortitle=search_list[i].select('.agent-color-listing')
                    if len(colortitle)<>0: #test if property name is color listed
                        colortitle=colortitle[0].select('a[title]')[0].get('title')
                        propertyinfo=[colortitle]+propertyinfo  
                        agentname=search_list[i].select('.agent-name')[0].getText()
                        price=search_list[i].select('.price')[0].getText()
                    else:
                        articleright_text=search_list[i].select('.article-right')[0].getText().splitlines()
                        agentname=articleright_text[2]
                        price=articleright_text[1]
                    
                    if len(propertyinfo)<7: #if less than 7 means the [Furnishing] info is missing
                        propertyinfo=propertyinfo[:4]+['Furnishing:Na']+propertyinfo[4:]
                    
                    contact_string=search_list[i].select('.con-number')[0].get('onclick') #the contact number buried inside this long string
                    contact=contact_string.split(',')[-3]
                    contact=contact.replace('\'','')
                    contact=contact.replace(' ','')
                                    
                    dic['name'].append(propertyinfo[0])
                    dic['loc'].append(propertyinfo[1])
                    dic['built_up'].append(propertyinfo[3])
                    dic['posted'].append(propertyinfo[5])
                    dic['type'].append(propertyinfo[2])
                    dic['furnish'].append(propertyinfo[4])
                    dic['price'].append(price)
                    dic['agent'].append(agentname)
                    dic['contact'].append(contact)
        except:
            print pg_num,i,contact           
    
    if found:
        table=df(dic)
        f_split=lambda x: x.split(':')[-1].strip()
        table['furnish']=table['furnish'].apply(f_split)
        table['furnish'][(table['furnish']=='Na')|(table['furnish']=='Unknown')]=np.nan
        table['posted']=table['posted'].apply(lambda x:':'.join(x.split(':')[1:]).strip())
        
        table['built_up']=table['built_up'].apply(extract_area)
        table['price']=table['price'].apply(extract_price)
        table['unit_price']=table['price'].div(table['built_up'])
        return table,i
    else:
        print "{} not found".format(keyword)

def urlreq(path):
    req=urllib2.Request(path)
    req.add_header('User-agent','Mozilla 3.10')
    res=urllib2.urlopen(req)
    html=res.read()
    return res,html

def request(path,ua):
    random.shuffle(ua)
    headers3={'User-Agent':random.choice(ua).strip()[1:-1]}
    headers='Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/600.7.12 (KHTML, like Gecko) Version/8.0.7 Safari/600.7.12'
    head={'User-Agent':headers}
    res=requests.get(path,headers=headers3)
    return res.text
    
def property_guru():
    with open('/Users/Daniel/Desktop/user_agents.txt','r') as f:
        ua=f.readlines()
    
    first='http://www.propertyguru.com.my/property-for-rent/'
    pg_num=1
    second='?limit=10&unselected=PROPERTY%7C361&market=residential&freetext='
    prop_name='danau murni'.replace(' ','+')
    dic={'name':[],'price':[],'area':[],'agent':[],'contact':[],'tenure':[],'time':[],'furnish':[],'url':[],'type':[]}
    wait_time=5
    for i in xrange(pg_num,100):
        T.sleep(wait_time*np.random.rand()+5) 
        path=first+str(i)+second+prop_name
        
        #set the user agent as the website will filter python user agent
        headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36'}
        headers2={'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.75.14 (KHTML, like Gecko) Version/7.0.3 Safari/7046A194A'}
       
        try:         
            res=request(path,ua)
        except:
            print 'Error URL'
            break
        
        
        soup=bs.BeautifulSoup(res)
        try:
            active=soup.select('div[class="listing-pagination"] li[class="active"]')[0].getText().strip()
        except:
            print i
            sys.exit()
            
        
        if int(active)==i:
            listing=soup.select('.listing-item')
            if len(listing)==0:
                print 'No search result'
                break
            found=True
            j=0
            for tag in listing:
                j+=1
                url=tag.select('h3 a')[0].get('href')
                name=tag.select('h3 a')[0].getText()
                price=tag.select('.listing-price')[0].getText()
                price=extract_price(price)
                area=tag.select('.lst-sizes')[0].getText()
                area=area.split('/')[0]
                area=extract_price(area)
                
                T.sleep(wait_time*np.random.rand()+5)
                
                
                item_url='http://www.propertyguru.com.my'+url
                res=request(item_url,ua)
                soup=bs.BeautifulSoup(res)
                try:
                    lefttd=soup.select('.col-sm-6')[0].select('td')
                except:
                    print i,j
                    sys.exit()
                    
                
                f=lambda x:np.nan if x=='-' else x
                types=lefttd[1].getText().strip()
                
                tenure=lefttd[3].getText().strip()
                
                time=lefttd[-1].getText().strip()
                boolean=True if 'days' in time else False
                
            
                righttd=soup.select('.col-sm-6')[-1].select('td')
                furnish=righttd[4].getText().strip()
                types,tenure,furnish=map(f,(types,tenure,furnish))
                agent=soup.select('.list-group-item-heading')[0].getText().strip()
                contact=soup.select('div[class="agent-phone"] span')[0].getText().strip()
    
                dic['name'].append(name)
                dic['price'].append(price)
                dic['area'].append(area)
                dic['type'].append(types)
                dic['tenure'].append(tenure)
                dic['agent'].append(agent)
                dic['contact'].append(contact)
                dic['time'].append(time)
                dic['url'].append('http://www.propertyguru.com.my'+url)
        
           
        else:
            break
        T.sleep(wait_time*np.random.rand()+1) 
    try:        
        if found:
            table=df(dic,columns=['name','area','price','type','tenure','time','agent','contact','url'])
            return table
        else:
            print 'Nothing is found'
    except:
        print 'Nothing is found'




