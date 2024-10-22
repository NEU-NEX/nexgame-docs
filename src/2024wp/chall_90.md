# 【中等】地图科技

刷到地图侦探的视频顺手出的，[台湾那期](https://www.bilibili.com/video/BV1rJsae2E2M/)，也是找风车，就改了下地点。

不过其实伦敦是个坑点，分成了伦敦城和伦敦城外围一圈两个部分，题目要求的是后者，最好去OpenStreetMap搜出来后，记下它的ID，（出题的时候差点直接写了个伦敦就放上去了XD）

服务端是这么计算的：

```python
def london_wind(request: Request, response: Response):
    import overpy
    api = overpy.Overpass()
    result = api.query("""[out:json][timeout:10];
area(id:3600065606)->.a;
nwr(area.a)["power"="generator"]["generator:source"="wind"]["generator:method"="wind_turbine"];
out geom;""")
    s = 0
    for i in result.nodes:
        s += i.id
        s %= 998244353
        
    return (len(result.nodes) * s)  % 998244353
```