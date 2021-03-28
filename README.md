# Warrents-arbitrage-opportunities-analysis

* 研究背景為華南永昌於2020年初因權證避險不及，出現巨虧。
* 主要原因為Open interest部位過大，且封關前不縮量，大量釋出價外(約20%-30%)具有套利機會的權證。
* 下圖為各權證部位示意圖，在台指期跌破價內時才停止出售。
![](https://i.imgur.com/6YaGcXa.png)

## 程式內容說明
* 主要分析華南永昌2020三月權證虧損背後之套利機會。
* 分別計算Open Interest最多的幾檔指數權證與相同到期日的TXO之IV和點數價差，及三月後之Greeks風險值。
* 可由下圖看出，華南權證(代號05786P)與到期日相同、履約價也相同的TXO約有1%的隱含波動度價差。

![](https://i.imgur.com/KRgmo8b.png)

* 計算IV之過程：若沒有相同到期日之可對應的期貨spot，則用時間插補法計算估計參數。
* 下方為mapping示意圖：

![示意圖](https://i.imgur.com/l9JYpNA.png)

* 三月VIX指數破50%，市場隱含波動度高，而券商未能及時反應價格時，容易拉大價差。

![](https://i.imgur.com/XcvdYxG.png)

* 隱含波動度：使用牛頓法來計算Black Scholes
* Code:

```python=
def implied_vol_newton(price_market, S, K, r, T, flag):
    
    if flag == 'call':
        n = 0
        sigma = 0.5
        diff = abs(price_market - bs_call(S, K, T, r, sigma))
        imp_vol = []
        bs_diff = []
        
        while (abs(diff) > 0.000001):
            diff = price_market - bs_call(S, K, T, r, sigma)
            imp_vol.append(sigma)
            bs_diff.append(abs(diff))
            sigma = (diff)/est_vega(S, K, r, sigma, T) + sigma
            n += 1
            if math.isnan(sigma):
                # print('nan')
                break
            if n > 10e3:
                break
        if len(imp_vol) > 0:
            sigma = imp_vol[bs_diff.index(min(bs_diff))]
        
        if sigma == 0.5:
            sigma = 'nan'
            
        return sigma
    
    if flag == 'put':
        n = 0
        sigma = 0.5
        diff = abs(price_market - bs_put(S, K, T, r, sigma))
        imp_vol = []
        bs_diff = []
        
        while (abs(diff) > 0.000001):
            diff = price_market - bs_put(S, K, T, r, sigma)
            imp_vol.append(sigma)
            bs_diff.append(abs(diff))
            sigma = (diff)/est_vega(S, K, r, sigma, T) + sigma
            # print(est_vega(S, K, r, sigma, T))
            n += 1
            if math.isnan(sigma):
                # print('nan')
                break
            if n > 10e3:
                break
        if len(imp_vol) > 0:
            sigma = imp_vol[bs_diff.index(min(bs_diff))]
        
        if sigma == 0.5:
            sigma = 'nan'
            
        return sigma
```




## 結論
![](https://i.imgur.com/SkhY9ID.png)


* 華南在發行權證前期避險不足，導致後期承擔過高避險成本，且發行權證的數量超過風險可承擔之胃納量。
* 三月波動較大，不利sell side，最高一日需承受之Gamma近23億，相當於大台1200口。
* 交易的風險控管機制應該要有發行規模的控管、Vega風險額度的限制。
