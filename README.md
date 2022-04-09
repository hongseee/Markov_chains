# Markov_chains
## 참고
- 01.Marketing Channel Attribution with Markov Chains in Python
    - https://towardsdatascience.com/marketing-channel-attribution-with-markov-chains-in-python-part-2-the-complete-walkthrough-733c65b23323
- 02.Word prediction with Markov chains in Python
    -  https://python.plainenglish.io/word-prediction-with-markov-chains-in-python-d685eed4b0b3

- 03.구매내역 데이터로 다음 구매 상품 예측
    - 데이터셋 출처 : http://archive.ics.uci.edu/ml/datasets/Online+Retail


---
# 03.구매내역 데이터로 다음 구매 상품 예측
```python
import numpy as np
import pandas as pd
from tqdm import tqdm
```

# 데이터 로드 및 확인


```python
df = pd.read_excel('/data/online_retail_II.xlsx', sheet_name='Year 2010-2011')
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Invoice</th>
      <th>StockCode</th>
      <th>Description</th>
      <th>Quantity</th>
      <th>InvoiceDate</th>
      <th>Price</th>
      <th>Customer ID</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>536365</td>
      <td>85123A</td>
      <td>WHITE HANGING HEART T-LIGHT HOLDER</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>2.55</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>1</th>
      <td>536365</td>
      <td>71053</td>
      <td>WHITE METAL LANTERN</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>3.39</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2</th>
      <td>536365</td>
      <td>84406B</td>
      <td>CREAM CUPID HEARTS COAT HANGER</td>
      <td>8</td>
      <td>2010-12-01 08:26:00</td>
      <td>2.75</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>3</th>
      <td>536365</td>
      <td>84029G</td>
      <td>KNITTED UNION FLAG HOT WATER BOTTLE</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>3.39</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>4</th>
      <td>536365</td>
      <td>84029E</td>
      <td>RED WOOLLY HOTTIE WHITE HEART.</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>3.39</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.rename(columns={'Customer ID':'CustomerID'}, inplace=True)
```




```python
print('상품 확인')
print('상품 코드:', len(df.StockCode.unique()))
print('상품 이름:',len(df.Description.unique()))
```

    상품 확인
    상품 코드: 4070
    상품 이름: 4224



```python
for i, v in df.StockCode.value_counts().items():
    if len(df[df.StockCode==i]['Description'].unique()) != 1:
        print(f'{i}-{df[df.StockCode==i].Description.unique()}')
```

    85123A-['WHITE HANGING HEART T-LIGHT HOLDER' '?' 'wrongly marked carton 22804'
     'CREAM HANGING HEART T-LIGHT HOLDER']
    22423-['REGENCY CAKESTAND 3 TIER' 'faulty' 'damages']
    20725-['LUNCH BAG RED RETROSPOT' 'LUNCH BAG RED SPOTTY']
    84879-['ASSORTED COLOUR BIRD ORNAMENT' 'damaged']
    22720-['SET OF 3 CAKE TINS PANTRY DESIGN ' nan]
    22197-['SMALL POPCORN HOLDER' 'POPCORN HOLDER']
    22383-['LUNCH BAG SUKI  DESIGN ' 'LUNCH BAG SUKI DESIGN ']
    23203-['mailout' 'JUMBO BAG DOILEY PATTERNS' 'JUMBO BAG VINTAGE DOILEY '
     'JUMBO BAG VINTAGE DOILY ']
    POST-['POSTAGE' nan]
    22469-['HEART OF WICKER SMALL' nan]
    23298-['SPOTTY BUNTING' 'BUNTING , SPOTTY ']
    23209-['mailout' 'LUNCH BAG DOILEY PATTERN ' 'LUNCH BAG VINTAGE DOILEY '
     'LUNCH BAG VINTAGE DOILY ']
    23084-['RABBIT NIGHT LIGHT' nan 'temp adjustment'
     'allocate stock for dotcom orders ta'
     'add stock to allocate online orders' 'for online retail orders' 'Amazon'
     'website fixed']
    22726-['ALARM CLOCK BAKELIKE GREEN' nan]
    22139-['RETROSPOT TEA SET CERAMIC 11 PC ' nan 'amazon']
    22470-['HEART OF WICKER LARGE' nan]



    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)


```python
df[df.StockCode==22467]
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Invoice</th>
      <th>StockCode</th>
      <th>Description</th>
      <th>Quantity</th>
      <th>InvoiceDate</th>
      <th>Price</th>
      <th>Customer ID</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>372</th>
      <td>536401</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>5</td>
      <td>2010-12-01 11:21:00</td>
      <td>2.55</td>
      <td>15862.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2731</th>
      <td>536592</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>1</td>
      <td>2010-12-01 17:06:00</td>
      <td>5.06</td>
      <td>NaN</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>3048</th>
      <td>536592</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>3</td>
      <td>2010-12-01 17:06:00</td>
      <td>2.55</td>
      <td>NaN</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>3266</th>
      <td>536615</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>12</td>
      <td>2010-12-02 10:09:00</td>
      <td>2.55</td>
      <td>14047.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>4441</th>
      <td>536783</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>108</td>
      <td>2010-12-02 15:19:00</td>
      <td>2.10</td>
      <td>15061.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>534941</th>
      <td>581175</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>72</td>
      <td>2011-12-07 15:16:00</td>
      <td>2.10</td>
      <td>14646.0</td>
      <td>Netherlands</td>
    </tr>
    <tr>
      <th>536915</th>
      <td>C581228</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>-36</td>
      <td>2011-12-08 10:06:00</td>
      <td>2.10</td>
      <td>16019.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>537146</th>
      <td>581238</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>1</td>
      <td>2011-12-08 10:53:00</td>
      <td>4.96</td>
      <td>NaN</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>540320</th>
      <td>581476</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>6</td>
      <td>2011-12-09 08:48:00</td>
      <td>2.55</td>
      <td>12433.0</td>
      <td>Norway</td>
    </tr>
    <tr>
      <th>540925</th>
      <td>581492</td>
      <td>22467</td>
      <td>GUMBALL COAT RACK</td>
      <td>1</td>
      <td>2011-12-09 10:03:00</td>
      <td>4.96</td>
      <td>NaN</td>
      <td>United Kingdom</td>
    </tr>
  </tbody>
</table>
<p>731 rows × 8 columns</p>
</div>



- 상품 코드로 상품 구별
- 가입 고객 정보로만 분석 수행(Customer ID is not null)
- 취소 데이터 제외

# Data Preprocessing


```python
# 고객 데이터 추출
customer_trade = df[~df['CustomerID'].isna()]
```




```python
print('고객수:', len(customer_trade['CustomerID'].unique()))
```

    고객수: 4372



```python
customer_trade.sort_values(['CustomerID', 'InvoiceDate'], ascending=[True, True])
```



```python
customer_trade['Invoice'] = customer_trade['Invoice'].astype(str)
```

```python
# 주문 취소 데이터 제외 
cancel_df = customer_trade[customer_trade['Invoice'].str.contains('C')]
cancel_df
```



```python
c_trade_idx = []
for info in tqdm(cancel_df.itertuples()):
#     print(info)
    idx = customer_trade[(customer_trade['StockCode']==info.StockCode)&(customer_trade['Description']==info.Description)
             &(customer_trade['Quantity']==abs(info.Quantity))&(customer_trade['Price']==info.Price)
              &(customer_trade['CustomerID']==info.CustomerID)].index
    c_trade_idx.extend(idx)

#     break
```

    8905it [08:36, 17.25it/s]



```python
c_trade_idx
```




```python
print(len(c_trade_idx)+len(list(cancel_df.index)))
print(len(c_trade_idx + list(cancel_df.index)))
```

    14430
    14430



```python
drop_idx = c_trade_idx + list(cancel_df.index)
```


```python
customer_trade = customer_trade.drop(drop_idx)
customer_trade
```



```python
customer_trade = customer_trade.sort_values(['CustomerID', 'InvoiceDate'], ascending=[True, True])
customer_trade['Trade_order'] = customer_trade.groupby('CustomerID').cumcount()+1
customer_trade
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Invoice</th>
      <th>StockCode</th>
      <th>Description</th>
      <th>Quantity</th>
      <th>InvoiceDate</th>
      <th>Price</th>
      <th>CustomerID</th>
      <th>Country</th>
      <th>Trade_order</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14938</th>
      <td>537626</td>
      <td>85116</td>
      <td>BLACK CANDELABRA T-LIGHT HOLDER</td>
      <td>12</td>
      <td>2010-12-07 14:57:00</td>
      <td>2.10</td>
      <td>12347.0</td>
      <td>Iceland</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14939</th>
      <td>537626</td>
      <td>22375</td>
      <td>AIRLINE BAG VINTAGE JET SET BROWN</td>
      <td>4</td>
      <td>2010-12-07 14:57:00</td>
      <td>4.25</td>
      <td>12347.0</td>
      <td>Iceland</td>
      <td>2</td>
    </tr>
    <tr>
      <th>14940</th>
      <td>537626</td>
      <td>71477</td>
      <td>COLOUR GLASS. STAR T-LIGHT HOLDER</td>
      <td>12</td>
      <td>2010-12-07 14:57:00</td>
      <td>3.25</td>
      <td>12347.0</td>
      <td>Iceland</td>
      <td>3</td>
    </tr>
    <tr>
      <th>14941</th>
      <td>537626</td>
      <td>22492</td>
      <td>MINI PAINT SET VINTAGE</td>
      <td>36</td>
      <td>2010-12-07 14:57:00</td>
      <td>0.65</td>
      <td>12347.0</td>
      <td>Iceland</td>
      <td>4</td>
    </tr>
    <tr>
      <th>14942</th>
      <td>537626</td>
      <td>22771</td>
      <td>CLEAR DRAWER KNOB ACRYLIC EDWARDIAN</td>
      <td>12</td>
      <td>2010-12-07 14:57:00</td>
      <td>1.25</td>
      <td>12347.0</td>
      <td>Iceland</td>
      <td>5</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>392752</th>
      <td>570715</td>
      <td>22419</td>
      <td>LIPSTICK PEN RED</td>
      <td>12</td>
      <td>2011-10-12 10:23:00</td>
      <td>0.42</td>
      <td>18287.0</td>
      <td>United Kingdom</td>
      <td>66</td>
    </tr>
    <tr>
      <th>392753</th>
      <td>570715</td>
      <td>22866</td>
      <td>HAND WARMER SCOTTY DOG DESIGN</td>
      <td>12</td>
      <td>2011-10-12 10:23:00</td>
      <td>2.10</td>
      <td>18287.0</td>
      <td>United Kingdom</td>
      <td>67</td>
    </tr>
    <tr>
      <th>423939</th>
      <td>573167</td>
      <td>23264</td>
      <td>SET OF 3 WOODEN SLEIGH DECORATIONS</td>
      <td>36</td>
      <td>2011-10-28 09:29:00</td>
      <td>1.25</td>
      <td>18287.0</td>
      <td>United Kingdom</td>
      <td>68</td>
    </tr>
    <tr>
      <th>423940</th>
      <td>573167</td>
      <td>21824</td>
      <td>PAINTED METAL STAR WITH HOLLY BELLS</td>
      <td>48</td>
      <td>2011-10-28 09:29:00</td>
      <td>0.39</td>
      <td>18287.0</td>
      <td>United Kingdom</td>
      <td>69</td>
    </tr>
    <tr>
      <th>423941</th>
      <td>573167</td>
      <td>21014</td>
      <td>SWISS CHALET TREE DECORATION</td>
      <td>24</td>
      <td>2011-10-28 09:29:00</td>
      <td>0.29</td>
      <td>18287.0</td>
      <td>United Kingdom</td>
      <td>70</td>
    </tr>
  </tbody>
</table>
<p>392768 rows × 9 columns</p>
</div>




```python
customer_t_order = customer_trade.groupby('CustomerID')['StockCode'].aggregate(
                lambda x : x.unique().tolist()).reset_index()
customer_t_order
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CustomerID</th>
      <th>StockCode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12347.0</td>
      <td>[85116, 22375, 71477, 22492, 22771, 22772, 227...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12348.0</td>
      <td>[84992, 22951, 84991, 21213, 22616, 21981, 219...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>12349.0</td>
      <td>[23112, 23460, 21564, 21411, 21563, 22131, 221...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>12350.0</td>
      <td>[21908, 22412, 79066K, 79191C, 22348, 84086C, ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12352.0</td>
      <td>[21380, 22064, 21232, 22646, 22779, 22423, 226...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4321</th>
      <td>18280.0</td>
      <td>[82484, 22180, 22467, 22725, 22727, 22495, 223...</td>
    </tr>
    <tr>
      <th>4322</th>
      <td>18281.0</td>
      <td>[22037, 22716, 22028, 23007, 23008, 23209, 22467]</td>
    </tr>
    <tr>
      <th>4323</th>
      <td>18282.0</td>
      <td>[21270, 23187, 23295, 22089, 21108, 21109, 224...</td>
    </tr>
    <tr>
      <th>4324</th>
      <td>18283.0</td>
      <td>[22356, 20726, 22384, 22386, 20717, 20718, 850...</td>
    </tr>
    <tr>
      <th>4325</th>
      <td>18287.0</td>
      <td>[22755, 22754, 22753, 22756, 22758, 22757, 227...</td>
    </tr>
  </tbody>
</table>
<p>4326 rows × 2 columns</p>
</div>



# 사전 만들기


```python
lexicon = {}
```


```python
def update_lexicon(current, next_stock) -> None:
    """Add item to the lexicon.
    """

    # Add the input word to the lexicon if it in there yet.
    if current not in lexicon:
        lexicon.update({current: {next_stock: 1} })
        return

    # Recieve te probabilties of the input word.
    options = lexicon[current]

    # Check if the output word is in the propability list.
    if next_stock not in options:
        options.update({next_stock : 1})
    else:
        options.update({next_stock : options[next_stock] + 1})

    # Update the lexicon
    lexicon[current] = options
```


```python
customer_t = customer_t_order[customer_t_order['CustomerID']==12347]['StockCode'][0]

```





```python
for i in range(len(customer_t)-1):
    update_lexicon(customer_t[i], customer_t[i+1])
```


```python
customer_t_order
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CustomerID</th>
      <th>StockCode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12347.0</td>
      <td>[85116, 22375, 71477, 22492, 22771, 22772, 227...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12348.0</td>
      <td>[84992, 22951, 84991, 21213, 22616, 21981, 219...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>12349.0</td>
      <td>[23112, 23460, 21564, 21411, 21563, 22131, 221...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>12350.0</td>
      <td>[21908, 22412, 79066K, 79191C, 22348, 84086C, ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12352.0</td>
      <td>[21380, 22064, 21232, 22646, 22779, 22423, 226...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4321</th>
      <td>18280.0</td>
      <td>[82484, 22180, 22467, 22725, 22727, 22495, 223...</td>
    </tr>
    <tr>
      <th>4322</th>
      <td>18281.0</td>
      <td>[22037, 22716, 22028, 23007, 23008, 23209, 22467]</td>
    </tr>
    <tr>
      <th>4323</th>
      <td>18282.0</td>
      <td>[21270, 23187, 23295, 22089, 21108, 21109, 224...</td>
    </tr>
    <tr>
      <th>4324</th>
      <td>18283.0</td>
      <td>[22356, 20726, 22384, 22386, 20717, 20718, 850...</td>
    </tr>
    <tr>
      <th>4325</th>
      <td>18287.0</td>
      <td>[22755, 22754, 22753, 22756, 22758, 22757, 227...</td>
    </tr>
  </tbody>
</table>
<p>4326 rows × 2 columns</p>
</div>




```python
lexicon = {}
for info in tqdm(custmer_t_order.itertuples()):
#     print(info.StockCode)
    stock_order = info.StockCode
#     print(stock_order)
    for i in range(len(stock_order)-1):
        update_lexicon(stock_order[i], stock_order[i+1])

#     break
```

    4326it [00:00, 13209.32it/s]






```python
# 확률 계산
for word, transition in tqdm(lexicon.items()):
    transition = dict((key, value / sum(transition.values())) for key, value in transition.items())
    lexicon[word] = transition
```

    100%|██████████| 3648/3648 [00:00<00:00, 10677.79it/s]






```python
line = input('> ')
word = line.strip().split(' ')[-1]

if word not in lexicon:
    if int(word) in lexicon:
        word = int(word)
        options = lexicon[word]
        predicted = np.random.choice(list(options.keys()), p=list(options.values()))
        print(line + ' ' + predicted) 
        
    else: 
        print('Word not found')
else:
    options = lexicon[word]
    predicted = np.random.choice(list(options.keys()), p=list(options.values()))
    print(line + ' ' + predicted)
```

    > 84952C
    84952C 84952B



```python
def nextStock():
    line = input('> ')
    word = line.strip().split(' ')[-1]

    if word not in lexicon:
        if int(word) in lexicon:
            word = int(word)
            options = lexicon[word]
            predicted = np.random.choice(list(options.keys()), p=list(options.values()))
            print(line + ' ' + predicted) 

        else: 
            print('Word not found')
    else:
        options = lexicon[word]
        predicted = np.random.choice(list(options.keys()), p=list(options.values()))
        print(line + ' ' + predicted)    
```


```python
nextStock()
```

    > 84952C 84946
    84952C 84946 23240



```python

```
