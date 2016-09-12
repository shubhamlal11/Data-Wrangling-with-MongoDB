
# Data Wrangling in MongoDB
##### by Shubham Lal


# About the dataset
> * Map Area : Mumbai, India
> * Source : http://www.openstreetmap.org/
> * File Sizes : Compressed -  6.61 MB, Uncompressed - 92.2 MB


# Checking Problems and Inconsistency in Dataset

### _Importing the dataset and finding the types and number of unique tags._

```python
import xml.etree.cElementTree as ET
import pprint

dic = {}
filename = 'mumbai.osm'
for event, element in ET.iterparse(filename):
    if element.tag in dic:
        dic[element.tag] +=1
    else:
        dic[element.tag] = 1
pprint.pprint(dic)        
```

    {'bounds': 1,
     'member': 8444,
     'nd': 536351,
     'node': 470060,
     'osm': 1,
     'relation': 126,
     'tag': 112613,
     'way': 40396}
    
> There are total eight unique tags in the dataset.
> At this point of time, we are unaware about the parent and child tags.

###  _Calculating different types of value stored in attribute 'k' in "tag" element.

> "lower" consists of normal value.
> "lower_colon" contains ":" in them.
> "problemchars" contains characters other than alphanumeric.

```python
import re
import xml.etree.cElementTree as ET
import pprint
filename = 'mumbai.osm'

lower = re.compile(r'^([a-z]|_)*$')
lower_colon = re.compile(r'^([a-z]|_)*:([a-z]|_)*$')
problemchars = re.compile(r'[=\+/&<>;\'"\?%#$@\,\. \t\r\n]')

keys = {'lower' : 0, 'lower_colon' : 0, 'problemchars' : 0, 'other' : 0}

for event, element in ET.iterparse(filename):
    if element.tag == 'tag':
        d = element.attrib['k']
        flag = 0
        if(re.match(lower,d)):
            keys['lower']+=1
            flag = 1
        elif(re.match(lower_colon,d)):
            keys['lower_colon']+=1
            flag = 1
        
        if flag==0:
            for c in d:
                if(re.match(problemchars, c)):
                    keys['problemchars']+=1
                    flag = 2
                    break
                    
        if flag ==0:
            keys['other'] +=1
        pass

pprint.pprint(keys)
```

    {'lower': 108481, 'lower_colon': 3270, 'other': 842, 'problemchars': 20}
    
> Problemchars are in very small number.
> Hence, it can be rectified manually.

### _Finding the number of users, who contributed in the dataset.

```python
import re
import xml.etree.cElementTree as ET
import pprint
filename = 'mumbai.osm'

users = set()
for event, element in ET.iterparse(filename):
    try:
        users.add(element.attrib['uid'])
    except:
        pass
pprint.pprint(len(users))        
```

    462
    
> This is number of unique users who have contributed in the dataset.

### _Scanning the value contained in attribute 'k' of "tag" element which contains ":"_

```python
import re
import xml.etree.cElementTree as ET
import pprint
filename = 'mumbai.osm'

lower_colon = re.compile(r'^([a-z]|_)*:([a-z]|_)*$')

unique_colon_keys = set()

for event, element in ET.iterparse(filename):    
    if element.tag == "tag":
        d = element.attrib['k']
        if re.match(lower_colon,d):
            unique_colon_keys.add(d)
            
pprint.pprint(unique_colon_keys)
    
```

    set(['addr:city',
         'addr:country',
         'addr:full',
         'addr:housename',
         'addr:housenumber',
         'addr:postcode',
         'addr:state',
         'addr:street',
         'addr:unit',
         'building:levels',
         'fuel:biodiesel',
         'fuel:biogas',
         'fuel:cng',
         'fuel:diesel',
         'fuel:electricity',
         'fuel:lpg',
         'gns:dsg',
         'gns:uni',
         'internet_access:fee',
         'is_in:city',
         'is_in:continent',
         'is_in:country',
         'is_in:country_code',
         'is_in:county',
         'is_in:state',
         'name:',
         'name:bn',
         'name:cs',
         'name:de',
         'name:en',
         'name:es',
         'name:fr',
         'name:gu',
         'name:hi',
         'name:jbo',
         'name:kn',
         'name:ma',
         'name:mr',
         'name:pl',
         'name:pt',
         'name:ru',
         'name:sk',
         'name:sr',
         'name:ta',
         'name:te',
         'oneway:bicycle',
         'payment:bitcoin',
         'place:cca',
         'ref:new',
         'seamark:fixme',
         'seamark:longname',
         'seamark:name',
         'seamark:topmark',
         'seamark:type',
         'ship:type',
         'shop:type',
         'source:position',
         'source:tracer',
         'source:url',
         'source:zoomlevel',
         'tower:type',
         'turn:lanes',
         'wp:foot',
         'wp:highway',
         'wp:rampatbeg',
         'wp:rampatend'])
         

    
### _Creating a set of the keywords which are contained before ":"_ 

```python
import re
import xml.etree.cElementTree as ET
import pprint
filename = 'mumbai.osm'

lower_colon = re.compile(r'^([a-z]|_)*:([a-z]|_)*$')
colon_keys = set()

for event, element in ET.iterparse(filename):     
    if element.tag == "tag":
        d = element.attrib['k']
        if re.match(lower_colon,d):
            l = d.split(':')
            colon_keys.add(l[0])
pprint.pprint(colon_keys)            
           
            
```

    set(['addr',
         'building',
         'fuel',
         'gns',
         'internet_access',
         'is_in',
         'name',
         'oneway',
         'payment',
         'place',
         'ref',
         'seamark',
         'ship',
         'shop',
         'source',
         'tower',
         'turn',
         'wp'])
    
### _Calculating number of each of such keywords in dataset._

```python
import re
import xml.etree.cElementTree as ET
import pprint
filename = 'mumbai.osm'

lower_colon = re.compile(r'^([a-z]|_)*:([a-z]|_)*$')
names = {}

for event, element in ET.iterparse(filename):
    keys = names.keys()
    if element.tag == "tag":
        d = element.attrib['k']
        if re.match(lower_colon,d):
            l = d.split(':')
            if l[0] in keys:
                names[l[0]] +=1
            else:
                names[l[0]] = 1
pprint.pprint(names) 
```

    {'addr': 2155,
     'building': 28,
     'fuel': 31,
     'gns': 4,
     'internet_access': 3,
     'is_in': 24,
     'name': 241,
     'oneway': 1,
     'payment': 1,
     'place': 1,
     'ref': 462,
     'seamark': 136,
     'ship': 1,
     'shop': 21,
     'source': 153,
     'tower': 3,
     'turn': 1,
     'wp': 4}
    

### _Keywords which are common in both "lower" set and "lower_colon" set

```python
colon_keys_indiv = []
for event, element in ET.iterparse(filename):
    for child in element:
        if child.tag == 'tag':
            if child.attrib['k'] in colon_keys:
                if child.attrib['k'] not in colon_keys_indiv:
                    colon_keys_indiv.append(child.attrib['k'])
colon_keys_indiv.append('gns')
colon_keys_indiv.append('addr')

pprint.pprint(colon_keys_indiv)
```

    ['name',
     'place',
     'source',
     'is_in',
     'ref',
     'shop',
     'building',
     'internet_access',
     'oneway',
     'fuel',
     'gns',
     'addr']
    
### _Creating  a dictionary to replace the above keyword with another keyword_


```python
colon_keys_indiv_cov = ['Current_name', 'Place', 'Source', 'Is_in', 'Reference', 'Shop', 'Building', 'Internet_Access',
                       'Oneway', 'Fuel', 'GEOnet Name Server', 'Address']
colon_keys_map = {}
i = 0
while i < len(colon_keys_indiv_cov):
    colon_keys_map[colon_keys_indiv[i]] = colon_keys_indiv_cov[i]
    i+=1
    
pprint.pprint(colon_keys_map)    
```

    {'addr': 'Address',
     'building': 'Building',
     'fuel': 'Fuel',
     'gns': 'GEOnet Name Server',
     'internet_access': 'Internet_Access',
     'is_in': 'Is_in',
     'name': 'Current_name',
     'oneway': 'Oneway',
     'place': 'Place',
     'ref': 'Reference',
     'shop': 'Shop',
     'source': 'Source'}
    
### _Finding parent elements._

```python
import re
import xml.etree.cElementTree as ET
import pprint
filename = 'mumbai.osm'

s = set()
for  event, element in ET.iterparse(filename):
    if element.tag == 'osm':
        for child in element:
            s.add(child.tag)
pprint.pprint(s)            
```

    set(['bounds', 'node', 'relation', 'way'])
    
### _Finding child elements of each parent element._

```python
from collections import defaultdict
dict = defaultdict(set)

for event, element in ET.iterparse(filename):
    if element.tag in s:        
        for child in element:
            dict[element.tag].add(child.tag)
pprint.pprint(dict)            
```

    defaultdict(<type 'set'>, {'node': set(['tag']), 'relation': set(['member', 'tag']), 'way': set(['tag', 'nd'])})
    
### _List of all short forms used to display different names._

```python
name_list = []
for event, element in ET.iterparse(filename):
    if element.tag == 'tag':
        if 'name:' in element.attrib['k']:
            full_name = element.attrib['k']
            name_splt = full_name.split(':')
            if name_splt[1] not in name_list:
                name_list.append(name_splt[1])
pprint.pprint(name_list)                
```

    ['bn',
     'cs',
     'de',
     'en',
     'es',
     'fr',
     'gu',
     'hi',
     'jbo',
     'kn',
     'mr',
     'ru',
     'sk',
     'sr',
     'ta',
     'te',
     'pt',
     'ma',
     '',
     'pl']
    
### _Creating a list of abbreviation._

>This is the list of abbreviation of above mentioned short forms of languages.

```python
real_name_list = ['Bengali', 'Czech', 'German', 'English', 'Spanish', 'French', 'Gujrati', 'Hindi', 'Lobjan', 'Kanada',
                  'Marathi', 'Russian', 'Slovak', 'Serbian', 'Tamil', 'Telugu', 'Portuguese', 'Arabic', None, 'Polish']
if len(real_name_list) == len(name_list):
    print True
else:
    print False
```

    True
    
### _Matching short forms with their respective abbreviation._

```python
name_matches = {}
for i in range(len(real_name_list)):
    name_matches[name_list[i]] = real_name_list[i]
pprint.pprint(name_matches)    
```

    {'': None,
     'bn': 'Bengali',
     'cs': 'Czech',
     'de': 'German',
     'en': 'English',
     'es': 'Spanish',
     'fr': 'French',
     'gu': 'Gujrati',
     'hi': 'Hindi',
     'jbo': 'Lobjan',
     'kn': 'Kanada',
     'ma': 'Arabic',
     'mr': 'Marathi',
     'pl': 'Polish',
     'pt': 'Portuguese',
     'ru': 'Russian',
     'sk': 'Slovak',
     'sr': 'Serbian',
     'ta': 'Tamil',
     'te': 'Telugu'}
    
### _Understanding different formats of value stored in 'k' attribute._

```python
tag_keys = []
for event, element in ET.iterparse(filename):
    if element.tag == 'tag':
        if ':' not in element.attrib['k']:
            if element.attrib['k'] not in tag_keys:
                tag_keys.append(element.attrib['k'])
pprint.pprint(tag_keys)
```

    ['admin_level',
     'ele',
     'is_capital',
     'name',
     'old_name',
     'place',
     'population',
     'rank',
     'source',
     'aerodrome',
     'aeroway',
     'iata',
     'icao',
     'is_in',
     'passengers',
     'type',
     -
     -
     ---------OUTPUT HIDDEN---------
     
>The output seems to contain only alphabetic characters. Hence, no modification is required.
    
### _Checking for consistency in Postal Codes._

```python
postal_code_list = set()
postcode_list = set()
addr_postcode_list = set()
for event, element in ET.iterparse(filename):
    if element.tag == "tag":
        if element.attrib['k'] == 'postal_code':            
            postal_code_list.add(element.attrib['v'])
        elif element.attrib['k'] == 'addr:postcode':
            addr_postcode_list.add(element.attrib['v'])
        elif element.attrib['k'] == 'postcode':
            postcode_list.add(element.attrib['v'])
print "Postal Code List - \n\n"
pprint.pprint(postal_code_list)
print "Postcode List - \n\n"
pprint.pprint(postcode_list)
print "Addr:postcode List - \n\n"
pprint.pprint(addr_postcode_list)

```

    Postal Code List -     
    
    set(['400001',
         '400002',
         '400003',
         '400004',
         '400005',
         '400006',
         '400007',
         '400008',
         '400009',
         '400011',
         '400013',
         '400018',
         '400020',
         '400021',
         '400023',
         '400026',
         '40003',
         '400034',
         '400036',
         '40013',
         '400702',
         '401200',
         '401204',
         '402106',
         '402200',
         '402202',
         '402203',
         '402204',
         '410200',
         '410201',
         '410202',
         '410205',
         '410210',
         '410218',
         '410300',
         '410400',
         '410401',
         '421100',
         '421301',
         '421302',
         '421501',
         '421503',
         '421601',
         '421604'])
         
         
    Postcode List -   
    
    set(['400008'])
    
    
    Addr:postcode List - 
    
    
    set([' 410201',
         '400 022',
         '400 601',
         '400001',
         '400005',
         '400007',
         '400010',
         '400013',
         '400016',
         '400018',
         '400019',
         '400020',
         '400021',
         '400022',
         '400030',
         '400033',
         '400034',
         '400035',
         '400036',
         '400038',
         '400039',
         '400043',
         '400047',
         '400049',
         '400050',
         '400052',
         '400053',
         '400054',
         '400056',
         '400057',
         '400058',
         '4000607',
         '400061',
         '400062',
         '400063',
         '400064',
         '400066',
         '400067',
         '400068',
         '400069',
         '400071',
         '400072',
         '400074',
         '400076',
         '400076, India',
         '400077',
         '400078',
         '400080',
         '400086',
         '400087',
         '400088',
         '400089',
         '400091',
         '400093',
         '400096',
         '400098',
         '400101',
         '400102',
         '400103',
         '40049',
         '40058',
         '400601',
         '400606',
         '400607',
         '400610',
         '400614',
         '400705',
         '400706',
         '401104 ',
         '401202',
         '401203',
         '401303',
         '410 201',
         '410210',
         '410701',
         '421202',
         '421501',
         '421503',
         '63103'])
         
>There are few values in the above list which is inconsistent with the dataset. They are: 
'40003','40013','400 022','400 601','4000607','400076, India',40049',40058','410 201','63103'

>Since, the number of such inconsistent data is less, it can be rectified manually.
    

### _Checking wheather any 'Node' or 'Way' contains two or more of postal code data_

```python
l = 0
ans = 0
ls = []
for event, element in ET.iterparse(filename):
    if element in ['node', 'way']:
        l = 0
        ls = []
        for child in element:
            if child.tag == 'tag':
                if child.attrib['k'] in ['postal_code', 'addr:postcode', 'postcode']:
                    ls.append(child.attrib['v'])
        if len(ls) == 3:
            ans+=1
print ans
```

    0
    
>This implies that each Node and Way element contains only one of three attributes (postal_code_, postcode, addr:postcode) containing the information about the postal code. 
    
### _Cleaning postal code data_

```python
postal_code_map = {}
postal_code_map['40003'] = '400003'
postal_code_map['40013'] = '400013'
postal_code_map['4000607'] = '400607'
postal_code_map['40049'] = '400049'
postal_code_map['40058'] = '400058'
pprint.pprint(postal_code_map)
```

    {'40003': '400003',
     '4000607': '400607',
     '40013': '400013',
     '40049': '400049',
     '40058': '400058'}
    

### _Attributes of Node tag_

```python
node_attrib = []
for event, element in ET.iterparse(filename):
    if element.tag == 'node':
        for attrib in element.attrib:
            if attrib not in node_attrib:
                node_attrib.append(attrib)
pprint.pprint(node_attrib)            
        
```

    ['changeset', 'uid', 'timestamp', 'lon', 'version', 'user', 'lat', 'id']
    

### _Attributes of Way tag_

```python
way_attrib = []
for event, element in ET.iterparse(filename):
    if element.tag == 'way':
        for attrib in element.attrib:
            if attrib not in way_attrib:
                way_attrib.append(attrib)
pprint.pprint(way_attrib)   
```

    ['changeset', 'uid', 'timestamp', 'version', 'user', 'id']
    
# Shaping data in python dictionary

### _Structure of the desired dictionary_

```
  { 
      id : , 
      type : , 
      visible : , 
      created : { 
                 version : , 
                 changeset : , 
                 timestamp : , 
                 user : , 
                 uid : 
               }, 
      pos : [latitude, longitude], 
      address : { 
                 housenumber: , 
                 postcode : , 
                 street : 
                }, 
      amenity : , 
      cuisine : , 
      name : , 
      phone : , 
      . 
      . 
      _other fields_ 
      . 
      . 
    }
```    
    
> Now we can start putting the data from xml to a python dictionary and then make a list of dictionaries that can be stored into a json file. While creating the dictionary, we shall take care of the problems we detected earlier.

```python
def insert_tag(child, indiv):
    name_list = child.attrib['k'].split(":")
    colon_name  = name_list[0]
    key = child.attrib['k'].replace(colon_name + ':', '')
    
    if colon_name == 'name':
            key = name_matches[key]    
            if key == None:
                return
    temp = {key : child.attrib['v']}
    
    try:           
        indiv[colon_name][key] = child.attrib['v']
    except:
        indiv[colon_name] = temp

        
final_list = []

for event, element in ET.iterparse(filename):
        
    if element.tag in ['way', 'node']:
        indiv = {}
        pos = []
        ref = []
        indiv['Element'] = element.tag
        for attrib in element.attrib:
            
            if attrib in ['version', 'changeset', 'user', 'uid', 'timestamp']:
                try:
                    indiv['created'][attrib] = element.attrib[attrib]
                except:
                    indiv['created'] = {}
                    indiv['created'][attrib] = element.attrib[attrib]           
                    
            elif attrib == 'lat':
                pos.insert(0, element.attrib['lat'])
                if len(pos) == 2:
                    indiv['pos'] = pos
            elif attrib == 'lon':
                pos.append(float(element.attrib['lon']))
                if len(pos) == 2:
                    indiv['pos'] = pos
                    
            else:
                indiv[attrib] = element.attrib[attrib]
        
        for child in element:
            if child.tag in ['tag', 'nd']:
                
                if child.tag == 'member':
                    insert_member(child, indiv)
                
                if child.tag == 'nd':                    
                    ref.append(child.attrib['ref'])                    
                    if(ref):
                        indiv['ref'] = ref
                
                if child.tag == 'tag':                    
                        
                    if child.attrib['k'] in ['postal_code', 'addr:postcode', 'postcode']:
                        if child.attrib['v'] in postal_code_map.keys():
                            postal_code_clean = postal_code_map[child.attrib['v']]
                            
                        elif ' ' in child.attrib['v']:
                            postal_code_clean = child.attrib['v']
                            postal_code_clean = postal_code_clean.replace(' ','')
                                
                        elif ',' in child.attrib['v']:
                            postal_code_clean_ls = child.attrib['v'].split(',')
                            postal_code_clean = postal_code_clean_ls[0]
                                
                        else:
                            postal_code_clean = child.attrib['v']
                            
                        indiv['Postal Code'] = postal_code_clean
                            
                    elif child.attrib['k'] == 'population':
                        indiv['Population'] = float(child.attrib['v'])                            
                        
                    elif child.attrib['k'] in colon_keys_map.keys():
                        indiv[colon_keys_map[child.attrib['k']]] = child.attrib['v']
                            
                    elif ":" in child.attrib['k']:
                        insert_tag(child, indiv)
                        
                    else:
                        indiv[child.attrib['k']] = child.attrib['v']    
                    
            
         
        if(indiv):
            final_list.append(indiv)            
            

```

### _Inserting the generated list in JSON file_ 

```python
import json
with open('data-wrangling-mongo-db.json', 'w') as final:
    json.dump(final_list, final)
```

>The JSON dataset will be stored in 'data-wrangling-mongo-db.json' file in the same directory.


# Inserting the data in MongoDB


```python
import pymongo
import json
connection = pymongo.MongoClient("mongodb://localhost")
db = connection.osm_data
record = db.mumbai_data
mumbai_data  = open('data-wrangling-mongo-db.json', 'r')
parsed_mumbai_data = json.loads(mumbai_data.read())

for entry in parsed_mumbai_data:
    record.insert_one(entry)
```

### _Number of records_

```python
record.find().count()
```

    510456


### _First five records of the data_

```python
import pprint
i = 0
for data in record.find():
    pprint.pprint(data)
    print '\n\n\n'
    i+=1
    if i == 5:
        break
```

    {u'Current_name': u'Mumbai',
     u'Element': u'node',
     u'Place': u'city',
     u'Population': 13662885.0,
     u'Source': u'http://en.wikipedia.org/wiki/Mumbai',
     u'_id': ObjectId('57c703d52da86022440d198c'),
     u'admin_level': u'4',
     u'created': {u'changeset': u'18660349',
                  u'timestamp': u'2013-11-01T22:11:09Z',
                  u'uid': u'1306',
                  u'user': u'PlaneMad',
                  u'version': u'31'},
     u'ele': u'8',
     u'id': u'16173235',
     u'is_capital': u'state',
     u'is_in': {u'continent': u'Asia',
                u'country': u'India',
                u'country_code': u'IN',
                u'state': u'Maharashtra'},
     u'name': {u'Bengali': u'\u09ae\u09c1\u09ae\u09cd\u09ac\u0987',
               u'Czech': u'Bombaj',
               u'English': u'Mumbai',
               u'French': u'Bombay',
               u'German': u'Mumbai',
               u'Gujrati': u'\u0aae\u0ac1\u0a82\u0aac\u0a88',
               u'Hindi': u'\u092e\u0941\u0902\u092c\u0908',
               u'Kanada': u'\u0cae\u0cc1\u0c82\u0cac\u0cc8',
               u'Lobjan': u'.mumbais.',
               u'Marathi': u'\u092e\u0941\u0902\u092c\u0908',
               u'Russian': u'\u041c\u0443\u043c\u0431\u0430\u0438',
               u'Serbian': u'\u041c\u0443\u043c\u0431\u0430\u0458',
               u'Slovak': u'Bombaj',
               u'Spanish': u'Bombay',
               u'Tamil': u'\u0bae\u0bc1\u0bae\u0bcd\u0baa\u0bc8',
               u'Telugu': u'\u0c2e\u0c41\u0c02\u0c2c\u0c48'},
     u'old_name': u'Bombay',
     u'place': {u'cca': u'a1'},
     u'pos': [u'18.9523804', 72.8327112],
     u'rank': u'0'}
    
    
    
    
    {u'Current_name': u'Chhatrapati Shivaji International Airport',
     u'Element': u'node',
     u'Is_in': u'Mumbai, Maharashtra,India',
     u'Source': u'Gagravarr_Airports',
     u'_id': ObjectId('57c703d72da86022440d198d'),
     u'aerodrome': u'international',
     u'aeroway': u'aerodrome',
     u'created': {u'changeset': u'12375819',
                  u'timestamp': u'2012-07-20T12:24:48Z',
                  u'uid': u'16135',
                  u'user': u'nimix',
                  u'version': u'11'},
     u'iata': u'BOM',
     u'icao': u'VABB',
     u'id': u'26609008',
     u'is_in': {u'country': u'India'},
     u'passengers': u'28137717',
     u'pos': [u'19.0895145', 72.8650825],
     u'type': u'civil'}
    
    
    
    
    {u'Element': u'node',
     u'_id': ObjectId('57c703d72da86022440d198e'),
     u'created': {u'changeset': u'3225476',
                  u'timestamp': u'2009-11-27T05:01:03Z',
                  u'uid': u'17497',
                  u'user': u'katpatuka',
                  u'version': u'5'},
     u'id': u'30517892',
     u'pos': [u'18.9795707', 73.0774771]}
    
    
    
    
    {u'Element': u'node',
     u'_id': ObjectId('57c703d72da86022440d198f'),
     u'created': {u'changeset': u'3225476',
                  u'timestamp': u'2009-11-27T05:01:04Z',
                  u'uid': u'17497',
                  u'user': u'katpatuka',
                  u'version': u'5'},
     u'id': u'30517895',
     u'pos': [u'18.9821882', 73.0882167]}
    
    
    
    
    {u'Element': u'node',
     u'_id': ObjectId('57c703d72da86022440d1990'),
     u'created': {u'changeset': u'545199',
                  u'timestamp': u'2008-01-05T16:32:19Z',
                  u'uid': u'1164',
                  u'user': u'dmgroom',
                  u'version': u'4'},
     u'created_by': u'JOSM',
     u'id': u'30517898',
     u'pos': [u'18.9777895', 73.0971373]}
    
    
    
    
### _Number of records which contains a value of "Amenity"_


```python
record.find({"$or" : [{"amenity" : {"$exists" : 1}}, {"Amenity" : {"$exists" : 1}}]}).count()
```

    1985


### _Different amenities in Mumbai with their count_

```python
amenity_data = record.aggregate([
        {"$group" : {"_id" : "$amenity",
                    "count" : {"$sum" : 1}}},
        {"$sort" : {"count" : -1}}
    ])
for doc in amenity_data:
    pprint.pprint(doc)
```

    {u'_id': None, u'count': 508472}
    {u'_id': u'place_of_worship', u'count': 246}
    {u'_id': u'school', u'count': 211}
    {u'_id': u'restaurant', u'count': 161}
    {u'_id': u'bank', u'count': 134}
    {u'_id': u'parking', u'count': 117}
    {u'_id': u'hospital', u'count': 110}
    {u'_id': u'bus_station', u'count': 102}
    {u'_id': u'fuel', u'count': 100}
    {u'_id': u'college', u'count': 83}
    {u'_id': u'fast_food', u'count': 66}
    {u'_id': u'police', u'count': 57}
    {u'_id': u'cafe', u'count': 55}
    {u'_id': u'atm', u'count': 51}
    {u'_id': u'cinema', u'count': 50}
    {u'_id': u'swimming_pool', u'count': 48}
    {u'_id': u'post_office', u'count': 40}
    {u'_id': u'pharmacy', u'count': 36}
    {u'_id': u'toilets', u'count': 35}
    {u'_id': u'marketplace', u'count': 27}
    {u'_id': u'public_building', u'count': 22}
    {u'_id': u'fire_station', u'count': 21}
    {u'_id': u'theatre', u'count': 17}
    {u'_id': u'post_box', u'count': 14}
    {u'_id': u'ferry_terminal', u'count': 13}
    {u'_id': u'parking;fuel', u'count': 12}
    {u'_id': u'university', u'count': 12}
    {u'_id': u'taxi', u'count': 12}
    {u'_id': u'community_centre', u'count': 11}
    {u'_id': u'library', u'count': 10}
    {u'_id': u'bar', u'count': 9}
    {u'_id': u'clinic', u'count': 8}
    {u'_id': u'pub', u'count': 7}
    {u'_id': u'bench', u'count': 6}
    {u'_id': u'courthouse', u'count': 6}
    {u'_id': u'fountain', u'count': 5}
    {u'_id': u'drinking_water', u'count': 5}
    {u'_id': u'townhall', u'count': 5}
    {u'_id': u'prison', u'count': 4}
    {u'_id': u'kindergarten', u'count': 4}
    {u'_id': u'food_court', u'count': 4}
    {u'_id': u'restroom', u'count': 4}
    {u'_id': u'grave_yard', u'count': 4}
    {u'_id': u'telephone', u'count': 3}
    {u'_id': u'vending_machine', u'count': 2}
    {u'_id': u'gym', u'count': 2}
    {u'_id': u'bicycle_rental', u'count': 2}
    {u'_id': u'car_wash', u'count': 2}
    {u'_id': u'fuel; kiosk', u'count': 2}
    {u'_id': u'market', u'count': 1}
    {u'_id': u'doctors', u'count': 1}
    {u'_id': u'cyber_cafe', u'count': 1}
    {u'_id': u'ice_cream', u'count': 1}
    {u'_id': u'shelter', u'count': 1}
    {u'_id': u'studio', u'count': 1}
    {u'_id': u'crematorium', u'count': 1}
    {u'_id': u'tea_shop', u'count': 1}
    {u'_id': u'Society', u'count': 1}
    {u'_id': u'arts_centre', u'count': 1}
    {u'_id': u'club', u'count': 1}
    {u'_id': u'water', u'count': 1}
    {u'_id': u'shop', u'count': 1}
    {u'_id': u'cold_storage', u'count': 1}
    {u'_id': u'picnic spot', u'count': 1}
    {u'_id': u'bureau_de_change', u'count': 1}
    {u'_id': u'nightclub', u'count': 1}
    {u'_id': u'waste_transfer_station', u'count': 1}
    {u'_id': u'pl', u'count': 1}
    {u'_id': u'fairgrounds', u'count': 1}
    {u'_id': u'social_centre', u'count': 1}
    {u'_id': u'social_facility', u'count': 1}
    {u'_id': u'Educational Complex', u'count': 1}
    {u'_id': u'Canteen', u'count': 1}
    {u'_id': u'Workshop', u'count': 1}
    {u'_id': u'creamatorium', u'count': 1}
    {u'_id': u'Gymkhana', u'count': 1}
    

### _Number of different sites for tourism in Mumbai_

```python
tourism_data = record.aggregate([
        {"$group" : {"_id" : "$tourism",
                    "count" : {"$sum" : 1}}},
        {"$sort" : {"count" : -1}}
    ])
for doc in tourism_data:
    pprint.pprint(doc)
```

    {u'_id': None, u'count': 510176}
    {u'_id': u'hotel', u'count': 181}
    {u'_id': u'hostel', u'count': 24}
    {u'_id': u'attraction', u'count': 24}
    {u'_id': u'viewpoint', u'count': 23}
    {u'_id': u'guest_house', u'count': 10}
    {u'_id': u'museum', u'count': 7}
    {u'_id': u'theme_park', u'count': 4}
    {u'_id': u'Byramjee Jeejeebhoy Point', u'count': 1}
    {u'_id': u'The Wall', u'count': 1}
    {u'_id': u'picnic_site', u'count': 1}
    {u'_id': u'information', u'count': 1}
    {u'_id': u'motel', u'count': 1}
    {u'_id': u'camp_site', u'count': 1}
    {u'_id': u'zoo', u'count': 1}
    

### _Records which has a value for "Population"

```python
record.find({"Population" : {"$exists" : 1}}).count()
```

    19


### _List of top 10 most active users on Mumbai Open Street Map_

```python
user_data = record.aggregate([
        {"$group" : {"_id" : "$created.user",
                    "count" : {"$sum" : 1}}},
        {"$sort" : {"count" : -1}},
        {"$limit" : 20}
    ])
for user in user_data:
    pprint.pprint(user)
```

    {u'_id': u'PlaneMad', u'count': 65571}
    {u'_id': u'MJL Wood', u'count': 61257}
    {u'_id': u'balaji88', u'count': 59941}
    {u'_id': u'parambyte', u'count': 45287}
    {u'_id': u'udaya', u'count': 44663}
    {u'_id': u'smith_dsm', u'count': 39375}
    {u'_id': u'Giyavudeen', u'count': 30007}
    {u'_id': u'indigomc', u'count': 19682}
    {u'_id': u'shekhar', u'count': 13296}
    {u'_id': u'Moorthy1', u'count': 11864}
    {u'_id': u'singleton', u'count': 10324}
    {u'_id': u'PremK', u'count': 10180}
    {u'_id': u'dmgroom_coastlines', u'count': 7880}
    {u'_id': u'gaurav jain', u'count': 7754}
    {u'_id': u'Heinz_V', u'count': 6907}
    {u'_id': u'Oberaffe', u'count': 6393}
    {u'_id': u'Meghanand', u'count': 6135}
    {u'_id': u'Shekhar11', u'count': 4717}
    {u'_id': u'jain zachariah', u'count': 4545}
    {u'_id': u'katpatuka', u'count': 4402}
    
### _Most followed releigions in Mumbai_

```python
religion_data = record.aggregate([
        {"$group" : {"_id" : "$religion",
                    "count" : {"$sum" : 1}}},
        {"$sort" : {"count" : -1}}
    ])
for doc in religion_data:
    pprint.pprint(doc)
```

    {u'_id': None, u'count': 510239}
    {u'_id': u'hindu', u'count': 82}
    {u'_id': u'muslim', u'count': 55}
    {u'_id': u'christian', u'count': 47}
    {u'_id': u'zoroastrian', u'count': 11}
    {u'_id': u'jewish', u'count': 7}
    {u'_id': u'jain', u'count': 6}
    {u'_id': u'sikh', u'count': 5}
    {u'_id': u'buddhist', u'count': 2}
    {u'_id': u'Jain', u'count': 1}
    {u'_id': u'hare_krishna', u'count': 1}
    

### _Creating a sample JSON Dataset_

```python
counter = 0
sample_list = []
for dictry in final_list:
    sample_list.append(dictry)
    counter+=10
    if counter >= len(final_list):
        break
```
>Storing every 10th entry of the original dataset in the sample dataset.

### _Number of entries in sample document_

```python
len(sample_list)
```

    51046


### _Insering the sample dataset in file_

```python
with open('data-wrangling-mongo-db-sample.json', 'w') as sample:
    json.dump(sample_list, sample)
```
>The sample dataset will be stored in 'data-wrangling-mongo-db-sample.json' file in the same directory.

## Final JSON dataset

* Complete JSON file : 107.0 MB
* Sample JSON file : 10.3 MB
