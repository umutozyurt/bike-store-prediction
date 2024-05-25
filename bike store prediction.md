```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import sum, year, month
import pyspark.sql.functions as F
```


```python
# Spark oturumu oluşturun
spark = SparkSession.builder.appName("UrunAnaliz").getOrCreate()
```


```python
# CSV dosyalarını okuma
brands = spark.read.csv("C:\\Users\\Umut\\Desktop\\brands.csv", header=True, inferSchema=True)
categories = spark.read.csv("C:\\Users\\Umut\\Desktop\\categories.csv", header=True, inferSchema=True)
customers = spark.read.csv("C:\\Users\\Umut\\Desktop\\customers.csv", header=True, inferSchema=True)
orders = spark.read.csv("C:\\Users\\Umut\\Desktop\\orders.csv", header=True, inferSchema=True)
staffs = spark.read.csv("C:\\Users\\Umut\\Desktop\\staffs.csv", header=True, inferSchema=True)
stores = spark.read.csv("C:\\Users\\Umut\\Desktop\\stores.csv", header=True, inferSchema=True)
orders_items = spark.read.csv("C:\\Users\\Umut\\Desktop\\order_items.csv", header=True, inferSchema=True)
stocks = spark.read.csv("C:\\Users\\Umut\\Desktop\\stocks.csv", header=True, inferSchema=True)
products = spark.read.csv("C:\\Users\\Umut\\Desktop\\products.csv", header=True, inferSchema=True)
```


```python
# Gerektiğinde tarih sütunlarını datetime formatına çevirin
# Örneğin, orders veri setindeki order_date sütununu datetime formatına çevirebilirsiniz.
```


```python
# 2016-2018 yılları arasındaki siparişleri filtreleyin
filtered_orders = orders.filter((orders.order_date >= '2016-01-01') & (orders.order_date <= '2018-12-31'))
```


```python
# Siparişler ve ürünler veri setlerini birleştirin
merged_data = (
    filtered_orders
    .join(orders_items, on="order_id")
    .join(products, on="product_id")
    .withColumn("year", year("order_date"))
    .withColumn("month", month("order_date"))
)
```


```python
# Ürünleri yıl ve ay bazında toplam satış miktarına göre gruplayın
product_sales = (
    merged_data
    .groupBy("year", "month", "product_name")
    .agg(sum("quantity").alias("total_sales"))
)
```


```python
# Satışları azalan sırada sıralayın
product_sales = product_sales.orderBy("year", "month", F.desc("total_sales"))
```


```python
# Tüm satırları göstermek için
product_sales.show(product_sales.count(), truncate=False)
```

    +----+-----+-----------------------------------------------------+-----------+
    |year|month|product_name                                         |total_sales|
    +----+-----+-----------------------------------------------------+-----------+
    |2016|1    |Electra Townie Original 7D EQ - 2016                 |21         |
    |2016|1    |Electra Cruiser 1 (24-Inch) - 2016                   |21         |
    |2016|1    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |20         |
    |2016|1    |Trek Remedy 29 Carbon Frameset - 2016                |14         |
    |2016|1    |Surly Straggler - 2016                               |14         |
    |2016|1    |Electra Townie Original 7D EQ - Women's - 2016       |12         |
    |2016|1    |Electra Townie Original 21D - 2016                   |12         |
    |2016|1    |Trek Slash 8 27.5 - 2016                             |12         |
    |2016|1    |Surly Wednesday Frameset - 2016                      |11         |
    |2016|1    |Pure Cycles Vine 8-Speed - 2016                      |11         |
    |2016|1    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |10         |
    |2016|1    |Trek Conduit+ - 2016                                 |10         |
    |2016|1    |Trek Fuel EX 8 29 - 2016                             |9          |
    |2016|1    |Heller Shagamaw Frame - 2016                         |8          |
    |2016|1    |Electra Townie Original 7D - 2015/2016               |7          |
    |2016|1    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |6          |
    |2016|1    |Electra Moto 1 - 2016                                |5          |
    |2016|1    |Ritchey Timberwolf Frameset - 2016                   |5          |
    |2016|1    |Pure Cycles William 3-Speed - 2016                   |5          |
    |2016|1    |Surly Ice Cream Truck Frameset - 2016                |4          |
    |2016|1    |Surly Straggler 650b - 2016                          |4          |
    |2016|2    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |23         |
    |2016|2    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |21         |
    |2016|2    |Electra Townie Original 7D - 2015/2016               |21         |
    |2016|2    |Electra Cruiser 1 (24-Inch) - 2016                   |19         |
    |2016|2    |Electra Townie Original 21D - 2016                   |13         |
    |2016|2    |Electra Townie Original 7D EQ - 2016                 |13         |
    |2016|2    |Electra Townie Original 7D EQ - Women's - 2016       |12         |
    |2016|2    |Electra Moto 1 - 2016                                |11         |
    |2016|2    |Pure Cycles Vine 8-Speed - 2016                      |10         |
    |2016|2    |Surly Wednesday Frameset - 2016                      |10         |
    |2016|2    |Pure Cycles William 3-Speed - 2016                   |9          |
    |2016|2    |Trek Remedy 29 Carbon Frameset - 2016                |9          |
    |2016|2    |Surly Straggler - 2016                               |8          |
    |2016|2    |Surly Ice Cream Truck Frameset - 2016                |7          |
    |2016|2    |Heller Shagamaw Frame - 2016                         |7          |
    |2016|2    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |6          |
    |2016|2    |Surly Straggler 650b - 2016                          |6          |
    |2016|2    |Trek Slash 8 27.5 - 2016                             |5          |
    |2016|2    |Ritchey Timberwolf Frameset - 2016                   |5          |
    |2016|2    |Trek Fuel EX 8 29 - 2016                             |4          |
    |2016|2    |Trek Conduit+ - 2016                                 |4          |
    |2016|3    |Electra Townie Original 21D - 2016                   |29         |
    |2016|3    |Electra Townie Original 7D EQ - 2016                 |24         |
    |2016|3    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |20         |
    |2016|3    |Surly Wednesday Frameset - 2016                      |13         |
    |2016|3    |Electra Cruiser 1 (24-Inch) - 2016                   |13         |
    |2016|3    |Surly Ice Cream Truck Frameset - 2016                |11         |
    |2016|3    |Trek Fuel EX 8 29 - 2016                             |10         |
    |2016|3    |Pure Cycles William 3-Speed - 2016                   |10         |
    |2016|3    |Ritchey Timberwolf Frameset - 2016                   |9          |
    |2016|3    |Trek Conduit+ - 2016                                 |9          |
    |2016|3    |Surly Straggler 650b - 2016                          |8          |
    |2016|3    |Heller Shagamaw Frame - 2016                         |8          |
    |2016|3    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |8          |
    |2016|3    |Pure Cycles Vine 8-Speed - 2016                      |7          |
    |2016|3    |Trek Slash 8 27.5 - 2016                             |7          |
    |2016|3    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |6          |
    |2016|3    |Electra Moto 1 - 2016                                |6          |
    |2016|3    |Electra Townie Original 7D - 2015/2016               |6          |
    |2016|3    |Electra Townie Original 7D EQ - Women's - 2016       |4          |
    |2016|3    |Trek Remedy 29 Carbon Frameset - 2016                |3          |
    |2016|3    |Surly Straggler - 2016                               |2          |
    |2016|4    |Electra Townie Original 21D - 2016                   |27         |
    |2016|4    |Electra Cruiser 1 (24-Inch) - 2016                   |14         |
    |2016|4    |Surly Straggler - 2016                               |13         |
    |2016|4    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |11         |
    |2016|4    |Trek Fuel EX 8 29 - 2016                             |11         |
    |2016|4    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |11         |
    |2016|4    |Electra Townie Original 7D EQ - Women's - 2016       |10         |
    |2016|4    |Surly Straggler 650b - 2016                          |10         |
    |2016|4    |Trek Slash 8 27.5 - 2016                             |9          |
    |2016|4    |Electra Townie Original 7D EQ - 2016                 |8          |
    |2016|4    |Ritchey Timberwolf Frameset - 2016                   |8          |
    |2016|4    |Surly Wednesday Frameset - 2016                      |8          |
    |2016|4    |Trek Remedy 29 Carbon Frameset - 2016                |7          |
    |2016|4    |Heller Shagamaw Frame - 2016                         |6          |
    |2016|4    |Electra Moto 1 - 2016                                |5          |
    |2016|4    |Electra Townie Original 7D - 2015/2016               |4          |
    |2016|4    |Surly Ice Cream Truck Frameset - 2016                |4          |
    |2016|4    |Pure Cycles William 3-Speed - 2016                   |4          |
    |2016|4    |Pure Cycles Vine 8-Speed - 2016                      |3          |
    |2016|4    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |3          |
    |2016|5    |Electra Cruiser 1 (24-Inch) - 2016                   |18         |
    |2016|5    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |18         |
    |2016|5    |Electra Townie Original 7D - 2015/2016               |15         |
    |2016|5    |Electra Townie Original 7D EQ - 2016                 |15         |
    |2016|5    |Electra Townie Original 21D - 2016                   |14         |
    |2016|5    |Trek Slash 8 27.5 - 2016                             |13         |
    |2016|5    |Pure Cycles Vine 8-Speed - 2016                      |13         |
    |2016|5    |Surly Straggler 650b - 2016                          |12         |
    |2016|5    |Surly Wednesday Frameset - 2016                      |11         |
    |2016|5    |Ritchey Timberwolf Frameset - 2016                   |11         |
    |2016|5    |Trek Remedy 29 Carbon Frameset - 2016                |10         |
    |2016|5    |Surly Straggler - 2016                               |10         |
    |2016|5    |Surly Ice Cream Truck Frameset - 2016                |10         |
    |2016|5    |Electra Moto 1 - 2016                                |8          |
    |2016|5    |Electra Townie Original 7D EQ - Women's - 2016       |8          |
    |2016|5    |Trek Fuel EX 8 29 - 2016                             |8          |
    |2016|5    |Heller Shagamaw Frame - 2016                         |8          |
    |2016|5    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |7          |
    |2016|5    |Pure Cycles William 3-Speed - 2016                   |7          |
    |2016|5    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |5          |
    |2016|5    |Trek Conduit+ - 2016                                 |3          |
    |2016|6    |Electra Cruiser 1 (24-Inch) - 2016                   |17         |
    |2016|6    |Trek Slash 8 27.5 - 2016                             |16         |
    |2016|6    |Electra Townie Original 7D EQ - 2016                 |16         |
    |2016|6    |Surly Ice Cream Truck Frameset - 2016                |15         |
    |2016|6    |Surly Straggler 650b - 2016                          |14         |
    |2016|6    |Ritchey Timberwolf Frameset - 2016                   |13         |
    |2016|6    |Trek Conduit+ - 2016                                 |11         |
    |2016|6    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |10         |
    |2016|6    |Electra Townie Original 21D - 2016                   |10         |
    |2016|6    |Surly Wednesday Frameset - 2016                      |9          |
    |2016|6    |Surly Straggler - 2016                               |8          |
    |2016|6    |Electra Townie Original 7D - 2015/2016               |8          |
    |2016|6    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |8          |
    |2016|6    |Pure Cycles William 3-Speed - 2016                   |7          |
    |2016|6    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |7          |
    |2016|6    |Pure Cycles Vine 8-Speed - 2016                      |6          |
    |2016|6    |Heller Shagamaw Frame - 2016                         |6          |
    |2016|6    |Electra Townie Original 7D EQ - Women's - 2016       |5          |
    |2016|6    |Trek Fuel EX 8 29 - 2016                             |5          |
    |2016|6    |Trek Remedy 29 Carbon Frameset - 2016                |4          |
    |2016|6    |Electra Moto 1 - 2016                                |4          |
    |2016|7    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |21         |
    |2016|7    |Electra Townie Original 21D - 2016                   |20         |
    |2016|7    |Electra Cruiser 1 (24-Inch) - 2016                   |17         |
    |2016|7    |Electra Townie Original 7D EQ - 2016                 |16         |
    |2016|7    |Surly Straggler - 2016                               |13         |
    |2016|7    |Surly Ice Cream Truck Frameset - 2016                |11         |
    |2016|7    |Surly Straggler 650b - 2016                          |11         |
    |2016|7    |Trek Conduit+ - 2016                                 |10         |
    |2016|7    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |10         |
    |2016|7    |Trek Slash 8 27.5 - 2016                             |10         |
    |2016|7    |Ritchey Timberwolf Frameset - 2016                   |9          |
    |2016|7    |Trek Fuel EX 8 29 - 2016                             |8          |
    |2016|7    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |8          |
    |2016|7    |Trek Remedy 29 Carbon Frameset - 2016                |7          |
    |2016|7    |Pure Cycles Vine 8-Speed - 2016                      |7          |
    |2016|7    |Heller Shagamaw Frame - 2016                         |7          |
    |2016|7    |Electra Moto 1 - 2016                                |7          |
    |2016|7    |Electra Townie Original 7D - 2015/2016               |7          |
    |2016|7    |Surly Wednesday Frameset - 2016                      |6          |
    |2016|7    |Electra Townie Original 7D EQ - Women's - 2016       |4          |
    |2016|7    |Pure Cycles William 3-Speed - 2016                   |2          |
    |2016|8    |Electra Cruiser 1 (24-Inch) - 2016                   |29         |
    |2016|8    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |27         |
    |2016|8    |Electra Townie Original 21D - 2016                   |21         |
    |2016|8    |Electra Townie Original 7D EQ - 2016                 |18         |
    |2016|8    |Surly Straggler - 2016                               |13         |
    |2016|8    |Trek Slash 8 27.5 - 2016                             |12         |
    |2016|8    |Electra Townie Original 7D - 2015/2016               |11         |
    |2016|8    |Trek Remedy 29 Carbon Frameset - 2016                |11         |
    |2016|8    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |11         |
    |2016|8    |Surly Ice Cream Truck Frameset - 2016                |10         |
    |2016|8    |Trek Fuel EX 8 29 - 2016                             |10         |
    |2016|8    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |9          |
    |2016|8    |Heller Shagamaw Frame - 2016                         |9          |
    |2016|8    |Surly Wednesday Frameset - 2016                      |9          |
    |2016|8    |Trek Conduit+ - 2016                                 |9          |
    |2016|8    |Electra Moto 1 - 2016                                |9          |
    |2016|8    |Ritchey Timberwolf Frameset - 2016                   |9          |
    |2016|8    |Surly Straggler 650b - 2016                          |8          |
    |2016|8    |Pure Cycles William 3-Speed - 2016                   |7          |
    |2016|8    |Electra Townie Original 7D EQ - Women's - 2016       |6          |
    |2016|8    |Pure Cycles Vine 8-Speed - 2016                      |3          |
    |2016|9    |Electra Cruiser 1 (24-Inch) - 2016                   |23         |
    |2016|9    |Electra Townie Original 7D EQ - 2016                 |22         |
    |2016|9    |Surly Ice Cream Truck Frameset - 2016                |20         |
    |2016|9    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |19         |
    |2016|9    |Electra Townie Original 21D - 2016                   |19         |
    |2016|9    |Trek Remedy 29 Carbon Frameset - 2016                |17         |
    |2016|9    |Trek Fuel EX 8 29 - 2016                             |17         |
    |2016|9    |Trek Slash 8 27.5 - 2016                             |15         |
    |2016|9    |Pure Cycles Vine 8-Speed - 2016                      |13         |
    |2016|9    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |12         |
    |2016|9    |Electra Moto 1 - 2016                                |12         |
    |2016|9    |Surly Wednesday Frameset - 2016                      |11         |
    |2016|9    |Electra Townie Original 7D EQ - Women's - 2016       |11         |
    |2016|9    |Surly Straggler - 2016                               |10         |
    |2016|9    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |10         |
    |2016|9    |Pure Cycles William 3-Speed - 2016                   |9          |
    |2016|9    |Trek Conduit+ - 2016                                 |9          |
    |2016|9    |Heller Shagamaw Frame - 2016                         |9          |
    |2016|9    |Ritchey Timberwolf Frameset - 2016                   |8          |
    |2016|9    |Surly Straggler 650b - 2016                          |8          |
    |2016|9    |Electra Townie Original 7D - 2015/2016               |7          |
    |2016|10   |Electra Cruiser 1 (24-Inch) - 2016                   |33         |
    |2016|10   |Electra Townie Original 21D - 2016                   |22         |
    |2016|10   |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |21         |
    |2016|10   |Electra Townie Original 7D EQ - 2016                 |20         |
    |2016|10   |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |16         |
    |2016|10   |Trek Conduit+ - 2016                                 |15         |
    |2016|10   |Electra Townie Original 7D - 2015/2016               |13         |
    |2016|10   |Surly Ice Cream Truck Frameset - 2016                |12         |
    |2016|10   |Pure Cycles Vine 8-Speed - 2016                      |10         |
    |2016|10   |Surly Straggler - 2016                               |10         |
    |2016|10   |Trek Fuel EX 8 29 - 2016                             |10         |
    |2016|10   |Ritchey Timberwolf Frameset - 2016                   |9          |
    |2016|10   |Surly Straggler 650b - 2016                          |9          |
    |2016|10   |Electra Moto 1 - 2016                                |9          |
    |2016|10   |Pure Cycles Western 3-Speed - Women's - 2015/2016    |8          |
    |2016|10   |Trek Remedy 29 Carbon Frameset - 2016                |8          |
    |2016|10   |Heller Shagamaw Frame - 2016                         |8          |
    |2016|10   |Electra Townie Original 7D EQ - Women's - 2016       |7          |
    |2016|10   |Trek Slash 8 27.5 - 2016                             |5          |
    |2016|10   |Pure Cycles William 3-Speed - 2016                   |5          |
    |2016|10   |Surly Wednesday Frameset - 2016                      |4          |
    |2016|11   |Electra Townie Original 7D EQ - 2016                 |20         |
    |2016|11   |Electra Cruiser 1 (24-Inch) - 2016                   |18         |
    |2016|11   |Electra Townie Original 21D - 2016                   |14         |
    |2016|11   |Trek Conduit+ - 2016                                 |13         |
    |2016|11   |Pure Cycles Western 3-Speed - Women's - 2015/2016    |11         |
    |2016|11   |Surly Straggler 650b - 2016                          |11         |
    |2016|11   |Trek Slash 8 27.5 - 2016                             |9          |
    |2016|11   |Electra Townie Original 7D - 2015/2016               |9          |
    |2016|11   |Electra Moto 1 - 2016                                |9          |
    |2016|11   |Trek Fuel EX 8 29 - 2016                             |9          |
    |2016|11   |Surly Wednesday Frameset - 2016                      |8          |
    |2016|11   |Pure Cycles William 3-Speed - 2016                   |7          |
    |2016|11   |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |6          |
    |2016|11   |Trek Remedy 29 Carbon Frameset - 2016                |6          |
    |2016|11   |Pure Cycles Vine 8-Speed - 2016                      |6          |
    |2016|11   |Electra Townie Original 7D EQ - Women's - 2016       |5          |
    |2016|11   |Surly Ice Cream Truck Frameset - 2016                |5          |
    |2016|11   |Ritchey Timberwolf Frameset - 2016                   |4          |
    |2016|11   |Heller Shagamaw Frame - 2016                         |4          |
    |2016|11   |Surly Straggler - 2016                               |4          |
    |2016|11   |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |3          |
    |2016|12   |Electra Townie Original 7D EQ - 2016                 |20         |
    |2016|12   |Pure Cycles Western 3-Speed - Women's - 2015/2016    |17         |
    |2016|12   |Electra Townie Original 21D - 2016                   |16         |
    |2016|12   |Electra Moto 1 - 2016                                |16         |
    |2016|12   |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |15         |
    |2016|12   |Electra Cruiser 1 (24-Inch) - 2016                   |15         |
    |2016|12   |Surly Straggler - 2016                               |14         |
    |2016|12   |Surly Straggler 650b - 2016                          |13         |
    |2016|12   |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |13         |
    |2016|12   |Heller Shagamaw Frame - 2016                         |13         |
    |2016|12   |Trek Conduit+ - 2016                                 |11         |
    |2016|12   |Trek Fuel EX 8 29 - 2016                             |10         |
    |2016|12   |Pure Cycles Vine 8-Speed - 2016                      |10         |
    |2016|12   |Trek Remedy 29 Carbon Frameset - 2016                |9          |
    |2016|12   |Electra Townie Original 7D - 2015/2016               |9          |
    |2016|12   |Pure Cycles William 3-Speed - 2016                   |8          |
    |2016|12   |Surly Ice Cream Truck Frameset - 2016                |7          |
    |2016|12   |Surly Wednesday Frameset - 2016                      |4          |
    |2016|12   |Ritchey Timberwolf Frameset - 2016                   |4          |
    |2016|12   |Trek Slash 8 27.5 - 2016                             |3          |
    |2016|12   |Electra Townie Original 7D EQ - Women's - 2016       |2          |
    |2017|1    |Sun Bicycles Cruz 3 - 2017                           |9          |
    |2017|1    |Trek Emonda S 4 - 2017                               |6          |
    |2017|1    |Electra Moto 1 - 2016                                |6          |
    |2017|1    |Trek Fuel EX 5 27.5 Plus - 2017                      |6          |
    |2017|1    |Electra Townie Original 7D - 2017                    |6          |
    |2017|1    |"Electra Girl's Hawaii 1 16"" - 2017"                |6          |
    |2017|1    |Trek Remedy 9.8 - 2017                               |5          |
    |2017|1    |Surly Steamroller - 2017                             |5          |
    |2017|1    |Trek Powerfly 8 FS Plus - 2017                       |5          |
    |2017|1    |Surly Ice Cream Truck Frameset - 2016                |4          |
    |2017|1    |Sun Bicycles Cruz 7 - Women's - 2017                 |4          |
    |2017|1    |Trek Domane SL Disc Frameset - 2017                  |4          |
    |2017|1    |Trek Domane SLR 6 Disc - 2017                        |4          |
    |2017|1    |Trek Conduit+ - 2016                                 |4          |
    |2017|1    |Electra Savannah 3i (20-inch) - Girl's - 2017        |4          |
    |2017|1    |Trek X-Caliber 8 - 2017                              |4          |
    |2017|1    |Surly Ice Cream Truck Frameset - 2017                |4          |
    |2017|1    |Trek Stache 5 - 2017                                 |4          |
    |2017|1    |Electra Townie Original 7D - 2015/2016               |4          |
    |2017|1    |Haro Downtown 16 - 2017                              |4          |
    |2017|1    |Electra Townie Original 7D EQ - 2016                 |4          |
    |2017|1    |Ritchey Timberwolf Frameset - 2016                   |4          |
    |2017|1    |Sun Bicycles Streamway - 2017                        |4          |
    |2017|1    |Electra Cruiser 1 (24-Inch) - 2016                   |4          |
    |2017|1    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |4          |
    |2017|1    |Electra Straight 8 3i (20-inch) - Boy's - 2017       |4          |
    |2017|1    |Surly Wednesday Frameset - 2017                      |3          |
    |2017|1    |Trek Boy's Kickster - 2015/2017                      |3          |
    |2017|1    |Pure Cycles Vine 8-Speed - 2016                      |3          |
    |2017|1    |Trek Domane S 6 - 2017                               |3          |
    |2017|1    |Trek Boone 7 - 2017                                  |3          |
    |2017|1    |Haro Shredder Pro 20 - 2017                          |3          |
    |2017|1    |Pure Cycles William 3-Speed - 2016                   |3          |
    |2017|1    |Trek Boone Race Shop Limited - 2017                  |3          |
    |2017|1    |Electra Townie Original 21D - 2016                   |3          |
    |2017|1    |Trek Slash 8 27.5 - 2016                             |3          |
    |2017|1    |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |3          |
    |2017|1    |Sun Bicycles Revolutions 24 - Girl's - 2017          |3          |
    |2017|1    |Trek Domane S 5 Disc - 2017                          |3          |
    |2017|1    |Surly Ogre Frameset - 2017                           |3          |
    |2017|1    |Sun Bicycles Drifter 7 - 2017                        |2          |
    |2017|1    |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |2          |
    |2017|1    |Heller Shagamaw Frame - 2016                         |2          |
    |2017|1    |Trek Precaliber 12 Girls - 2017                      |2          |
    |2017|1    |Trek Precaliber 24 (21-Speed) - Girls - 2017         |2          |
    |2017|1    |Trek Domane SL 6 - 2017                              |2          |
    |2017|1    |Sun Bicycles ElectroLite - 2017                      |2          |
    |2017|1    |Electra Amsterdam Fashion 7i Ladies' - 2017          |2          |
    |2017|1    |Trek Fuel EX 8 29 - 2016                             |2          |
    |2017|1    |Sun Bicycles Cruz 7 - 2017                           |2          |
    |2017|1    |Sun Bicycles Biscayne Tandem 7 - 2017                |2          |
    |2017|1    |Haro SR 1.2 - 2017                                   |2          |
    |2017|1    |Trek Silque SLR 8 Women's - 2017                     |2          |
    |2017|1    |Electra Amsterdam Original 3i - 2015/2017            |2          |
    |2017|1    |Electra Townie 7D (20-inch) - Boys' - 2017           |2          |
    |2017|1    |Trek Precaliber 16 Boys - 2017                       |2          |
    |2017|1    |Surly Wednesday Frameset - 2016                      |2          |
    |2017|1    |Haro Shift R3 - 2017                                 |2          |
    |2017|1    |Trek Silque SLR 7 Women's - 2017                     |2          |
    |2017|1    |Sun Bicycles Streamway 7 - 2017                      |2          |
    |2017|1    |Trek Farley Alloy Frameset - 2017                    |2          |
    |2017|1    |Surly Straggler 650b - 2016                          |2          |
    |2017|1    |Electra Moto 3i (20-inch) - Boy's - 2017             |2          |
    |2017|1    |Haro Flightline One ST - 2017                        |2          |
    |2017|1    |Sun Bicycles Revolutions 24 - 2017                   |1          |
    |2017|1    |Trek Madone 9.2 - 2017                               |1          |
    |2017|1    |Sun Bicycles Streamway 3 - 2017                      |1          |
    |2017|1    |Haro Flightline Two 26 Plus - 2017                   |1          |
    |2017|1    |Trek Precaliber 16 Girls - 2017                      |1          |
    |2017|1    |Electra Amsterdam Original 3i Ladies' - 2017         |1          |
    |2017|1    |Surly Karate Monkey 27.5+ Frameset - 2017            |1          |
    |2017|1    |Sun Bicycles Brickell Tandem 7 - 2017                |1          |
    |2017|1    |Sun Bicycles Cruz 3 - Women's - 2017                 |1          |
    |2017|1    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |1          |
    |2017|1    |Electra Cruiser Lux 1 - 2017                         |1          |
    |2017|1    |Sun Bicycles Lil Kitt'n - 2017                       |1          |
    |2017|1    |Electra Cruiser Lux Fat Tire 1 Ladies - 2017         |1          |
    |2017|1    |Sun Bicycles Atlas X-Type - 2017                     |1          |
    |2017|1    |Sun Bicycles Biscayne Tandem CB - 2017               |1          |
    |2017|1    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |1          |
    |2017|1    |Sun Bicycles Brickell Tandem CB - 2017               |1          |
    |2017|2    |Heller Shagamaw Frame - 2016                         |10         |
    |2017|2    |Trek Domane SLR 6 Disc - 2017                        |9          |
    |2017|2    |Haro SR 1.2 - 2017                                   |8          |
    |2017|2    |Trek Domane S 6 - 2017                               |8          |
    |2017|2    |Electra Amsterdam Original 3i Ladies' - 2017         |8          |
    |2017|2    |Trek Fuel EX 8 29 - 2016                             |8          |
    |2017|2    |Electra Townie Original 21D - 2016                   |7          |
    |2017|2    |Electra Townie Original 7D - 2017                    |7          |
    |2017|2    |Sun Bicycles Streamway - 2017                        |6          |
    |2017|2    |"Electra Girl's Hawaii 1 16"" - 2017"                |5          |
    |2017|2    |Haro Shredder 20 Girls - 2017                        |5          |
    |2017|2    |Trek Boone 7 - 2017                                  |5          |
    |2017|2    |Trek Precaliber 12 Girls - 2017                      |5          |
    |2017|2    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |5          |
    |2017|2    |Trek Emonda S 4 - 2017                               |5          |
    |2017|2    |Surly Wednesday Frameset - 2016                      |5          |
    |2017|2    |Electra Townie Original 7D EQ - 2016                 |5          |
    |2017|2    |Surly Straggler 650b - 2016                          |5          |
    |2017|2    |Pure Cycles William 3-Speed - 2016                   |5          |
    |2017|2    |Sun Bicycles Lil Kitt'n - 2017                       |5          |
    |2017|2    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |4          |
    |2017|2    |Electra Glam Punk 3i Ladies' - 2017                  |4          |
    |2017|2    |Electra Cruiser 1 (24-Inch) - 2016                   |4          |
    |2017|2    |Trek Fuel EX 9.8 29 - 2017                           |4          |
    |2017|2    |Surly Straggler - 2016                               |4          |
    |2017|2    |Sun Bicycles Cruz 3 - 2017                           |4          |
    |2017|2    |Electra Townie 7D (20-inch) - Boys' - 2017           |4          |
    |2017|2    |Haro Shredder 20 - 2017                              |4          |
    |2017|2    |Electra Moto 3i (20-inch) - Boy's - 2017             |3          |
    |2017|2    |Trek Powerfly 8 FS Plus - 2017                       |3          |
    |2017|2    |Trek Precaliber 24 (21-Speed) - Girls - 2017         |3          |
    |2017|2    |Electra Townie Original 7D - 2015/2016               |3          |
    |2017|2    |Trek Slash 8 27.5 - 2016                             |3          |
    |2017|2    |Sun Bicycles Drifter 7 - Women's - 2017              |3          |
    |2017|2    |Sun Bicycles Cruz 3 - Women's - 2017                 |3          |
    |2017|2    |Trek Stache 5 - 2017                                 |3          |
    |2017|2    |Sun Bicycles Biscayne Tandem 7 - 2017                |3          |
    |2017|2    |Trek Session DH 27.5 Carbon Frameset - 2017          |3          |
    |2017|2    |Trek Domane S 5 Disc - 2017                          |3          |
    |2017|2    |Surly Wednesday Frameset - 2017                      |3          |
    |2017|2    |Sun Bicycles ElectroLite - 2017                      |3          |
    |2017|2    |Trek Emonda S 5 - 2017                               |3          |
    |2017|2    |Haro SR 1.1 - 2017                                   |2          |
    |2017|2    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |2          |
    |2017|2    |Trek Boone Race Shop Limited - 2017                  |2          |
    |2017|2    |Sun Bicycles Revolutions 24 - 2017                   |2          |
    |2017|2    |Trek Madone 9.2 - 2017                               |2          |
    |2017|2    |Sun Bicycles Brickell Tandem 7 - 2017                |2          |
    |2017|2    |Sun Bicycles Cruz 7 - 2017                           |2          |
    |2017|2    |Trek Remedy 9.8 - 2017                               |2          |
    |2017|2    |Electra Cruiser Lux 1 - 2017                         |2          |
    |2017|2    |Sun Bicycles Lil Bolt Type-R - 2017                  |2          |
    |2017|2    |Electra Cruiser Lux Fat Tire 1 Ladies - 2017         |2          |
    |2017|2    |Electra Moto 1 - 2016                                |2          |
    |2017|2    |Sun Bicycles Cruz 7 - Women's - 2017                 |2          |
    |2017|2    |Sun Bicycles Atlas X-Type - 2017                     |2          |
    |2017|2    |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |2          |
    |2017|2    |Surly Wednesday - 2017                               |2          |
    |2017|2    |Haro Flightline Two 26 Plus - 2017                   |2          |
    |2017|2    |Sun Bicycles Streamway 3 - 2017                      |2          |
    |2017|2    |Surly Ice Cream Truck Frameset - 2016                |2          |
    |2017|2    |Haro Shredder Pro 20 - 2017                          |2          |
    |2017|2    |Sun Bicycles Revolutions 24 - Girl's - 2017          |2          |
    |2017|2    |Electra Savannah 3i (20-inch) - Girl's - 2017        |2          |
    |2017|2    |Surly Steamroller - 2017                             |1          |
    |2017|2    |Trek Precaliber 16 Girls - 2017                      |1          |
    |2017|2    |Surly Ogre Frameset - 2017                           |1          |
    |2017|2    |Trek Silque SLR 7 Women's - 2017                     |1          |
    |2017|2    |Trek Fuel EX 5 27.5 Plus - 2017                      |1          |
    |2017|2    |Electra Townie Original 7D EQ - Women's - 2016       |1          |
    |2017|2    |Surly Big Dummy Frameset - 2017                      |1          |
    |2017|2    |Pure Cycles Vine 8-Speed - 2016                      |1          |
    |2017|2    |Trek Boy's Kickster - 2015/2017                      |1          |
    |2017|2    |Electra Amsterdam Fashion 7i Ladies' - 2017          |1          |
    |2017|2    |Trek X-Caliber 8 - 2017                              |1          |
    |2017|2    |Surly Ice Cream Truck Frameset - 2017                |1          |
    |2017|2    |Haro Downtown 16 - 2017                              |1          |
    |2017|2    |Trek Domane SL 6 - 2017                              |1          |
    |2017|2    |Surly Karate Monkey 27.5+ Frameset - 2017            |1          |
    |2017|2    |Trek Remedy 29 Carbon Frameset - 2016                |1          |
    |2017|2    |Electra Amsterdam Original 3i - 2015/2017            |1          |
    |2017|2    |Sun Bicycles Biscayne Tandem CB - 2017               |1          |
    |2017|2    |Sun Bicycles Spider 3i - 2017                        |1          |
    |2017|3    |"Electra Girl's Hawaii 1 16"" - 2017"                |13         |
    |2017|3    |Electra Townie Original 7D EQ - 2016                 |11         |
    |2017|3    |Surly Ice Cream Truck Frameset - 2016                |9          |
    |2017|3    |Surly Wednesday Frameset - 2017                      |8          |
    |2017|3    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |8          |
    |2017|3    |Surly Troll Frameset - 2017                          |8          |
    |2017|3    |Trek Slash 8 27.5 - 2016                             |7          |
    |2017|3    |Electra Cruiser 1 (24-Inch) - 2016                   |7          |
    |2017|3    |Electra Townie Original 21D - 2016                   |7          |
    |2017|3    |Haro Shredder 20 Girls - 2017                        |6          |
    |2017|3    |Sun Bicycles Lil Bolt Type-R - 2017                  |6          |
    |2017|3    |Trek Boone 7 - 2017                                  |6          |
    |2017|3    |Trek X-Caliber 8 - 2017                              |6          |
    |2017|3    |Surly Karate Monkey 27.5+ Frameset - 2017            |6          |
    |2017|3    |Surly Steamroller - 2017                             |5          |
    |2017|3    |Pure Cycles Vine 8-Speed - 2016                      |5          |
    |2017|3    |Sun Bicycles Biscayne Tandem CB - 2017               |5          |
    |2017|3    |Electra Townie Original 7D - 2017                    |5          |
    |2017|3    |Trek Emonda S 5 - 2017                               |5          |
    |2017|3    |Haro Shift R3 - 2017                                 |5          |
    |2017|3    |Trek Precaliber 24 (21-Speed) - Girls - 2017         |5          |
    |2017|3    |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |5          |
    |2017|3    |Trek Madone 9.2 - 2017                               |4          |
    |2017|3    |Electra Townie Original 7D - 2015/2016               |4          |
    |2017|3    |Trek Domane SL Disc Frameset - 2017                  |4          |
    |2017|3    |Haro Shredder 20 - 2017                              |4          |
    |2017|3    |Trek Fuel EX 5 27.5 Plus - 2017                      |4          |
    |2017|3    |Trek Boy's Kickster - 2015/2017                      |4          |
    |2017|3    |Trek Precaliber 16 Girls - 2017                      |4          |
    |2017|3    |Electra Glam Punk 3i Ladies' - 2017                  |4          |
    |2017|3    |Sun Bicycles Brickell Tandem CB - 2017               |4          |
    |2017|3    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |3          |
    |2017|3    |Electra Townie Original 7D EQ - Women's - 2016       |3          |
    |2017|3    |Electra Cruiser Lux 1 - 2017                         |3          |
    |2017|3    |Sun Bicycles Spider 3i - 2017                        |3          |
    |2017|3    |Trek Conduit+ - 2016                                 |3          |
    |2017|3    |Surly Wednesday Frameset - 2016                      |3          |
    |2017|3    |Surly Straggler 650b - 2016                          |3          |
    |2017|3    |Trek Fuel EX 9.8 29 - 2017                           |3          |
    |2017|3    |Haro Downtown 16 - 2017                              |3          |
    |2017|3    |Pure Cycles William 3-Speed - 2016                   |3          |
    |2017|3    |Sun Bicycles Revolutions 24 - 2017                   |3          |
    |2017|3    |Sun Bicycles Streamway 3 - 2017                      |3          |
    |2017|3    |Sun Bicycles Cruz 7 - Women's - 2017                 |3          |
    |2017|3    |Surly Wednesday - 2017                               |3          |
    |2017|3    |Sun Bicycles Biscayne Tandem 7 - 2017                |3          |
    |2017|3    |Electra Amsterdam Fashion 7i Ladies' - 2017          |3          |
    |2017|3    |Heller Shagamaw Frame - 2016                         |3          |
    |2017|3    |Surly Ice Cream Truck Frameset - 2017                |2          |
    |2017|3    |Sun Bicycles Drifter 7 - Women's - 2017              |2          |
    |2017|3    |Trek Stache 5 - 2017                                 |2          |
    |2017|3    |Sun Bicycles Revolutions 24 - Girl's - 2017          |2          |
    |2017|3    |Sun Bicycles Streamway 7 - 2017                      |2          |
    |2017|3    |Electra Amsterdam Original 3i Ladies' - 2017         |2          |
    |2017|3    |Trek Boone Race Shop Limited - 2017                  |2          |
    |2017|3    |Ritchey Timberwolf Frameset - 2016                   |2          |
    |2017|3    |Trek Silque SLR 7 Women's - 2017                     |2          |
    |2017|3    |Trek Remedy 29 Carbon Frameset - 2016                |2          |
    |2017|3    |Electra Townie 7D (20-inch) - Boys' - 2017           |2          |
    |2017|3    |Sun Bicycles Drifter 7 - 2017                        |2          |
    |2017|3    |Trek Domane S 5 Disc - 2017                          |2          |
    |2017|3    |Surly Ogre Frameset - 2017                           |2          |
    |2017|3    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |2          |
    |2017|3    |Trek Session DH 27.5 Carbon Frameset - 2017          |2          |
    |2017|3    |Trek Silque SLR 8 Women's - 2017                     |2          |
    |2017|3    |Haro SR 1.2 - 2017                                   |2          |
    |2017|3    |Electra Moto 3i (20-inch) - Boy's - 2017             |2          |
    |2017|3    |Trek Farley Alloy Frameset - 2017                    |2          |
    |2017|3    |Trek Precaliber 16 Boys - 2017                       |2          |
    |2017|3    |Electra Cruiser Lux Fat Tire 1 Ladies - 2017         |2          |
    |2017|3    |Electra Savannah 3i (20-inch) - Girl's - 2017        |1          |
    |2017|3    |Trek Girl's Kickster - 2017                          |1          |
    |2017|3    |Sun Bicycles ElectroLite - 2017                      |1          |
    |2017|3    |Sun Bicycles Cruz 3 - 2017                           |1          |
    |2017|3    |Trek Precaliber 12 Girls - 2017                      |1          |
    |2017|3    |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |1          |
    |2017|3    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |1          |
    |2017|3    |Sun Bicycles Cruz 7 - 2017                           |1          |
    |2017|3    |Trek Domane S 6 - 2017                               |1          |
    |2017|3    |Surly Big Dummy Frameset - 2017                      |1          |
    |2017|3    |Sun Bicycles Atlas X-Type - 2017                     |1          |
    |2017|3    |Trek Remedy 9.8 - 2017                               |1          |
    |2017|3    |Haro Shredder Pro 20 - 2017                          |1          |
    |2017|3    |Trek Domane SLR 6 Disc - 2017                        |1          |
    |2017|3    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |1          |
    |2017|3    |Sun Bicycles Brickell Tandem 7 - 2017                |1          |
    |2017|3    |Trek Emonda S 4 - 2017                               |1          |
    |2017|4    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |9          |
    |2017|4    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |8          |
    |2017|4    |Haro SR 1.2 - 2017                                   |8          |
    |2017|4    |Electra Townie Original 7D EQ - 2016                 |8          |
    |2017|4    |Sun Bicycles Lil Bolt Type-R - 2017                  |7          |
    |2017|4    |Haro Downtown 16 - 2017                              |7          |
    |2017|4    |Haro Shredder 20 Girls - 2017                        |6          |
    |2017|4    |Sun Bicycles Streamway 7 - 2017                      |6          |
    |2017|4    |Trek Madone 9.2 - 2017                               |6          |
    |2017|4    |Sun Bicycles Streamway 3 - 2017                      |6          |
    |2017|4    |Electra Cruiser 1 (24-Inch) - 2016                   |5          |
    |2017|4    |Trek Emonda S 4 - 2017                               |5          |
    |2017|4    |Electra Townie Original 7D - 2017                    |5          |
    |2017|4    |Sun Bicycles Atlas X-Type - 2017                     |5          |
    |2017|4    |Electra Cruiser Lux Fat Tire 1 Ladies - 2017         |5          |
    |2017|4    |Trek Precaliber 12 Boys - 2017                       |5          |
    |2017|4    |Sun Bicycles Brickell Tandem 7 - 2017                |4          |
    |2017|4    |Trek Precaliber 12 Girls - 2017                      |4          |
    |2017|4    |Trek Boy's Kickster - 2015/2017                      |4          |
    |2017|4    |Haro SR 1.3 - 2017                                   |4          |
    |2017|4    |Haro Flightline Two 26 Plus - 2017                   |4          |
    |2017|4    |Sun Bicycles Cruz 7 - 2017                           |4          |
    |2017|4    |Electra Moto 3i (20-inch) - Boy's - 2017             |4          |
    |2017|4    |Sun Bicycles Biscayne Tandem 7 - 2017                |4          |
    |2017|4    |Electra Amsterdam Fashion 7i Ladies' - 2017          |4          |
    |2017|4    |"Electra Girl's Hawaii 1 16"" - 2017"                |4          |
    |2017|4    |Electra Townie Original 21D - 2016                   |4          |
    |2017|4    |Pure Cycles William 3-Speed - 2016                   |3          |
    |2017|4    |Sun Bicycles Cruz 3 - 2017                           |3          |
    |2017|4    |Haro Shift R3 - 2017                                 |3          |
    |2017|4    |Heller Shagamaw Frame - 2016                         |3          |
    |2017|4    |Trek Domane S 6 - 2017                               |3          |
    |2017|4    |Surly Ice Cream Truck Frameset - 2016                |3          |
    |2017|4    |Surly Troll Frameset - 2017                          |3          |
    |2017|4    |Trek Domane SLR 6 Disc - 2017                        |3          |
    |2017|4    |Sun Bicycles Cruz 7 - Women's - 2017                 |3          |
    |2017|4    |Sun Bicycles Cruz 3 - Women's - 2017                 |3          |
    |2017|4    |Surly Ice Cream Truck Frameset - 2017                |3          |
    |2017|4    |Electra Amsterdam Original 3i - 2015/2017            |3          |
    |2017|4    |Trek Fuel EX 5 27.5 Plus - 2017                      |3          |
    |2017|4    |Trek Domane SL Disc Frameset - 2017                  |3          |
    |2017|4    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |3          |
    |2017|4    |Sun Bicycles Revolutions 24 - Girl's - 2017          |3          |
    |2017|4    |Electra Townie Original 7D - 2015/2016               |3          |
    |2017|4    |Trek Precaliber 16 Girls - 2017                      |2          |
    |2017|4    |Surly Big Dummy Frameset - 2017                      |2          |
    |2017|4    |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |2          |
    |2017|4    |Surly Straggler 650b - 2016                          |2          |
    |2017|4    |Haro SR 1.1 - 2017                                   |2          |
    |2017|4    |Haro Flightline One ST - 2017                        |2          |
    |2017|4    |Trek Domane SL 6 - 2017                              |2          |
    |2017|4    |Surly Karate Monkey 27.5+ Frameset - 2017            |2          |
    |2017|4    |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |2          |
    |2017|4    |Surly Wednesday Frameset - 2017                      |2          |
    |2017|4    |Trek Powerfly 8 FS Plus - 2017                       |2          |
    |2017|4    |Trek Boone Race Shop Limited - 2017                  |2          |
    |2017|4    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |2          |
    |2017|4    |Electra Townie Original 7D EQ - Women's - 2016       |2          |
    |2017|4    |Haro Shredder 20 - 2017                              |2          |
    |2017|4    |Trek Girl's Kickster - 2017                          |2          |
    |2017|4    |Electra Townie 7D (20-inch) - Boys' - 2017           |2          |
    |2017|4    |Trek Remedy 9.8 - 2017                               |1          |
    |2017|4    |Trek Fuel EX 9.8 29 - 2017                           |1          |
    |2017|4    |Electra Amsterdam Original 3i Ladies' - 2017         |1          |
    |2017|4    |Surly Straggler - 2016                               |1          |
    |2017|4    |Trek Session DH 27.5 Carbon Frameset - 2017          |1          |
    |2017|4    |Sun Bicycles Revolutions 24 - 2017                   |1          |
    |2017|4    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |1          |
    |2017|4    |Sun Bicycles Brickell Tandem CB - 2017               |1          |
    |2017|4    |Sun Bicycles Biscayne Tandem CB - 2017               |1          |
    |2017|4    |Ritchey Timberwolf Frameset - 2016                   |1          |
    |2017|4    |Trek X-Caliber 8 - 2017                              |1          |
    |2017|4    |Trek Boone 7 - 2017                                  |1          |
    |2017|4    |Surly Ogre Frameset - 2017                           |1          |
    |2017|4    |Trek Precaliber 16 Boys - 2017                       |1          |
    |2017|4    |Trek Silque SLR 8 Women's - 2017                     |1          |
    |2017|4    |Electra Savannah 3i (20-inch) - Girl's - 2017        |1          |
    |2017|4    |Haro Shredder Pro 20 - 2017                          |1          |
    |2017|4    |Trek Conduit+ - 2016                                 |1          |
    |2017|5    |Sun Bicycles Cruz 7 - 2017                           |10         |
    |2017|5    |Electra Townie Original 7D EQ - 2016                 |8          |
    |2017|5    |Sun Bicycles Cruz 3 - 2017                           |8          |
    |2017|5    |Trek Silque SLR 7 Women's - 2017                     |7          |
    |2017|5    |Electra Townie Original 7D - 2017                    |7          |
    |2017|5    |Surly Karate Monkey 27.5+ Frameset - 2017            |6          |
    |2017|5    |Heller Shagamaw Frame - 2016                         |6          |
    |2017|5    |Haro SR 1.3 - 2017                                   |5          |
    |2017|5    |Trek Precaliber 24 (21-Speed) - Girls - 2017         |5          |
    |2017|5    |Sun Bicycles Revolutions 24 - 2017                   |5          |
    |2017|5    |Sun Bicycles Lil Bolt Type-R - 2017                  |5          |
    |2017|5    |Sun Bicycles Revolutions 24 - Girl's - 2017          |5          |
    |2017|5    |Electra Townie Original 21D - 2016                   |5          |
    |2017|5    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |5          |
    |2017|5    |Ritchey Timberwolf Frameset - 2016                   |5          |
    |2017|5    |Electra Cruiser 1 (24-Inch) - 2016                   |4          |
    |2017|5    |Trek Boone 7 - 2017                                  |4          |
    |2017|5    |Electra Townie 7D (20-inch) - Boys' - 2017           |4          |
    |2017|5    |Haro Shredder 20 - 2017                              |4          |
    |2017|5    |Surly Ice Cream Truck Frameset - 2016                |4          |
    |2017|5    |"Electra Girl's Hawaii 1 16"" - 2017"                |4          |
    |2017|5    |Haro Flightline Two 26 Plus - 2017                   |4          |
    |2017|5    |Trek Emonda S 4 - 2017                               |4          |
    |2017|5    |Trek Emonda S 5 - 2017                               |4          |
    |2017|5    |Surly Ice Cream Truck Frameset - 2017                |3          |
    |2017|5    |Sun Bicycles Streamway 7 - 2017                      |3          |
    |2017|5    |Electra Glam Punk 3i Ladies' - 2017                  |3          |
    |2017|5    |Trek Fuel EX 5 27.5 Plus - 2017                      |3          |
    |2017|5    |Sun Bicycles Brickell Tandem 7 - 2017                |3          |
    |2017|5    |Surly Wednesday - 2017                               |3          |
    |2017|5    |Trek Boone Race Shop Limited - 2017                  |3          |
    |2017|5    |Surly Wednesday Frameset - 2017                      |3          |
    |2017|5    |Trek Domane SL 6 - 2017                              |3          |
    |2017|5    |Sun Bicycles Cruz 3 - Women's - 2017                 |3          |
    |2017|5    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |3          |
    |2017|5    |Pure Cycles Vine 8-Speed - 2016                      |3          |
    |2017|5    |Electra Townie Original 7D EQ - Women's - 2016       |3          |
    |2017|5    |Haro SR 1.2 - 2017                                   |3          |
    |2017|5    |Surly Troll Frameset - 2017                          |3          |
    |2017|5    |Trek Farley Alloy Frameset - 2017                    |3          |
    |2017|5    |Trek Domane S 6 - 2017                               |2          |
    |2017|5    |Electra Moto 1 - 2016                                |2          |
    |2017|5    |Trek Powerfly 8 FS Plus - 2017                       |2          |
    |2017|5    |Trek Stache 5 - 2017                                 |2          |
    |2017|5    |Sun Bicycles Spider 3i - 2017                        |2          |
    |2017|5    |Surly Big Dummy Frameset - 2017                      |2          |
    |2017|5    |Trek Domane S 5 Disc - 2017                          |2          |
    |2017|5    |Trek Fuel EX 8 29 - 2016                             |2          |
    |2017|5    |Trek Boy's Kickster - 2015/2017                      |2          |
    |2017|5    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |2          |
    |2017|5    |Haro Shift R3 - 2017                                 |2          |
    |2017|5    |Electra Moto 3i (20-inch) - Boy's - 2017             |2          |
    |2017|5    |Sun Bicycles Lil Kitt'n - 2017                       |2          |
    |2017|5    |Sun Bicycles Streamway 3 - 2017                      |2          |
    |2017|5    |Trek Precaliber 16 Girls - 2017                      |2          |
    |2017|5    |Surly Straggler - 2016                               |2          |
    |2017|5    |Sun Bicycles Atlas X-Type - 2017                     |2          |
    |2017|5    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |2          |
    |2017|5    |Haro SR 1.1 - 2017                                   |2          |
    |2017|5    |Electra Cruiser Lux Fat Tire 1 Ladies - 2017         |2          |
    |2017|5    |Trek Silque SLR 8 Women's - 2017                     |2          |
    |2017|5    |Electra Cruiser Lux 1 - 2017                         |2          |
    |2017|5    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |2          |
    |2017|5    |Sun Bicycles Brickell Tandem CB - 2017               |2          |
    |2017|5    |Trek Slash 8 27.5 - 2016                             |2          |
    |2017|5    |Trek Conduit+ - 2016                                 |2          |
    |2017|5    |Electra Amsterdam Original 3i - 2015/2017            |1          |
    |2017|5    |Surly Steamroller - 2017                             |1          |
    |2017|5    |Electra Straight 8 3i (20-inch) - Boy's - 2017       |1          |
    |2017|5    |Electra Amsterdam Original 3i Ladies' - 2017         |1          |
    |2017|5    |Trek Precaliber 12 Girls - 2017                      |1          |
    |2017|5    |Trek Remedy 9.8 - 2017                               |1          |
    |2017|5    |Trek Domane SLR 6 Disc - 2017                        |1          |
    |2017|5    |Trek Precaliber 16 Boys - 2017                       |1          |
    |2017|5    |Sun Bicycles Biscayne Tandem CB - 2017               |1          |
    |2017|5    |Surly Ogre Frameset - 2017                           |1          |
    |2017|5    |Trek Girl's Kickster - 2017                          |1          |
    |2017|5    |Sun Bicycles Drifter 7 - Women's - 2017              |1          |
    |2017|5    |Haro Shredder Pro 20 - 2017                          |1          |
    |2017|6    |Sun Bicycles Lil Bolt Type-R - 2017                  |11         |
    |2017|6    |"Electra Girl's Hawaii 1 16"" - 2017"                |8          |
    |2017|6    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |8          |
    |2017|6    |Electra Townie Original 7D EQ - 2016                 |8          |
    |2017|6    |Electra Amsterdam Original 3i - 2015/2017            |7          |
    |2017|6    |Trek Domane SLR 6 Disc - 2017                        |7          |
    |2017|6    |Electra Townie Original 7D EQ - Women's - 2016       |6          |
    |2017|6    |Electra Cruiser 1 (24-Inch) - 2016                   |6          |
    |2017|6    |Trek Fuel EX 8 29 - 2016                             |6          |
    |2017|6    |Sun Bicycles Cruz 7 - 2017                           |6          |
    |2017|6    |Sun Bicycles Cruz 3 - Women's - 2017                 |6          |
    |2017|6    |Electra Townie Original 21D - 2016                   |6          |
    |2017|6    |Sun Bicycles Drifter 7 - 2017                        |5          |
    |2017|6    |Trek Boone 7 - 2017                                  |5          |
    |2017|6    |Surly Steamroller - 2017                             |5          |
    |2017|6    |Sun Bicycles Atlas X-Type - 2017                     |5          |
    |2017|6    |Trek Silque SLR 8 Women's - 2017                     |5          |
    |2017|6    |Electra Townie Original 7D - 2017                    |5          |
    |2017|6    |Trek Boone Race Shop Limited - 2017                  |5          |
    |2017|6    |Electra Cruiser Lux 1 - 2017                         |5          |
    |2017|6    |Trek Emonda S 4 - 2017                               |5          |
    |2017|6    |Haro Downtown 16 - 2017                              |5          |
    |2017|6    |Sun Bicycles Brickell Tandem 7 - 2017                |5          |
    |2017|6    |Sun Bicycles Cruz 3 - 2017                           |5          |
    |2017|6    |Sun Bicycles Spider 3i - 2017                        |5          |
    |2017|6    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |5          |
    |2017|6    |Electra Amsterdam Fashion 7i Ladies' - 2017          |4          |
    |2017|6    |Haro SR 1.1 - 2017                                   |4          |
    |2017|6    |Trek Domane SL 6 - 2017                              |4          |
    |2017|6    |Trek Remedy 9.8 - 2017                               |4          |
    |2017|6    |Trek Powerfly 8 FS Plus - 2017                       |4          |
    |2017|6    |Surly Troll Frameset - 2017                          |4          |
    |2017|6    |Surly Karate Monkey 27.5+ Frameset - 2017            |4          |
    |2017|6    |Electra Cruiser Lux Fat Tire 1 Ladies - 2017         |4          |
    |2017|6    |Electra Townie Original 7D - 2015/2016               |4          |
    |2017|6    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |4          |
    |2017|6    |Electra Glam Punk 3i Ladies' - 2017                  |3          |
    |2017|6    |Trek Silque SLR 7 Women's - 2017                     |3          |
    |2017|6    |Sun Bicycles Revolutions 24 - 2017                   |3          |
    |2017|6    |Trek Precaliber 12 Boys - 2017                       |3          |
    |2017|6    |Trek Precaliber 12 Girls - 2017                      |3          |
    |2017|6    |Electra Amsterdam Original 3i Ladies' - 2017         |3          |
    |2017|6    |Sun Bicycles Biscayne Tandem CB - 2017               |3          |
    |2017|6    |Trek Domane S 6 - 2017                               |3          |
    |2017|6    |Sun Bicycles Biscayne Tandem 7 - 2017                |3          |
    |2017|6    |Heller Shagamaw Frame - 2016                         |3          |
    |2017|6    |Electra Savannah 3i (20-inch) - Girl's - 2017        |3          |
    |2017|6    |Trek Boy's Kickster - 2015/2017                      |3          |
    |2017|6    |Trek Conduit+ - 2016                                 |3          |
    |2017|6    |Trek Precaliber 16 Girls - 2017                      |3          |
    |2017|6    |Trek Fuel EX 5 27.5 Plus - 2017                      |2          |
    |2017|6    |Haro Flightline One ST - 2017                        |2          |
    |2017|6    |Surly Straggler - 2016                               |2          |
    |2017|6    |Haro Shredder Pro 20 - 2017                          |2          |
    |2017|6    |Sun Bicycles Brickell Tandem CB - 2017               |2          |
    |2017|6    |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |2          |
    |2017|6    |Trek Domane SL Disc Frameset - 2017                  |2          |
    |2017|6    |Sun Bicycles Streamway 7 - 2017                      |2          |
    |2017|6    |Trek Remedy 29 Carbon Frameset - 2016                |2          |
    |2017|6    |Electra Straight 8 3i (20-inch) - Boy's - 2017       |2          |
    |2017|6    |Sun Bicycles Drifter 7 - Women's - 2017              |2          |
    |2017|6    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |2          |
    |2017|6    |Trek Session DH 27.5 Carbon Frameset - 2017          |2          |
    |2017|6    |Sun Bicycles Streamway 3 - 2017                      |2          |
    |2017|6    |Trek X-Caliber 8 - 2017                              |2          |
    |2017|6    |Trek Madone 9.2 - 2017                               |2          |
    |2017|6    |Surly Wednesday - 2017                               |2          |
    |2017|6    |Trek Fuel EX 9.8 29 - 2017                           |2          |
    |2017|6    |Surly Wednesday Frameset - 2017                      |2          |
    |2017|6    |Haro Flightline Two 26 Plus - 2017                   |2          |
    |2017|6    |Haro Shredder 20 - 2017                              |2          |
    |2017|6    |Surly Straggler 650b - 2016                          |2          |
    |2017|6    |Trek Domane S 5 Disc - 2017                          |2          |
    |2017|6    |Trek Farley Alloy Frameset - 2017                    |1          |
    |2017|6    |Trek Emonda S 5 - 2017                               |1          |
    |2017|6    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |1          |
    |2017|6    |Haro Shift R3 - 2017                                 |1          |
    |2017|6    |Surly Wednesday Frameset - 2016                      |1          |
    |2017|6    |Surly Ice Cream Truck Frameset - 2016                |1          |
    |2017|6    |Trek Slash 8 27.5 - 2016                             |1          |
    |2017|6    |Trek Stache 5 - 2017                                 |1          |
    |2017|6    |Electra Moto 1 - 2016                                |1          |
    |2017|6    |Sun Bicycles ElectroLite - 2017                      |1          |
    |2017|6    |Electra Moto 3i (20-inch) - Boy's - 2017             |1          |
    |2017|6    |Surly Big Dummy Frameset - 2017                      |1          |
    |2017|6    |Electra Townie 7D (20-inch) - Boys' - 2017           |1          |
    |2017|7    |"Electra Girl's Hawaii 1 16"" - 2017"                |11         |
    |2017|7    |Sun Bicycles Cruz 3 - 2017                           |10         |
    |2017|7    |Sun Bicycles Lil Bolt Type-R - 2017                  |8          |
    |2017|7    |Electra Townie Original 7D EQ - 2016                 |8          |
    |2017|7    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |8          |
    |2017|7    |Haro SR 1.2 - 2017                                   |6          |
    |2017|7    |Sun Bicycles ElectroLite - 2017                      |6          |
    |2017|7    |Surly Big Dummy Frameset - 2017                      |6          |
    |2017|7    |Electra Townie 7D (20-inch) - Boys' - 2017           |6          |
    |2017|7    |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |6          |
    |2017|7    |Electra Townie Original 7D EQ - Women's - 2016       |5          |
    |2017|7    |Trek Remedy 29 Carbon Frameset - 2016                |5          |
    |2017|7    |Surly Straggler - 2016                               |5          |
    |2017|7    |Electra Amsterdam Fashion 7i Ladies' - 2017          |4          |
    |2017|7    |Sun Bicycles Lil Kitt'n - 2017                       |4          |
    |2017|7    |Surly Troll Frameset - 2017                          |4          |
    |2017|7    |Trek Precaliber 24 (21-Speed) - Girls - 2017         |4          |
    |2017|7    |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |4          |
    |2017|7    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |4          |
    |2017|7    |Trek Silque SLR 8 Women's - 2017                     |4          |
    |2017|7    |Sun Bicycles Drifter 7 - Women's - 2017              |4          |
    |2017|7    |Sun Bicycles Cruz 7 - Women's - 2017                 |4          |
    |2017|7    |Surly Ice Cream Truck Frameset - 2016                |4          |
    |2017|7    |Electra Townie Original 7D - 2017                    |3          |
    |2017|7    |Trek Fuel EX 9.8 29 - 2017                           |3          |
    |2017|7    |Sun Bicycles Revolutions 24 - 2017                   |3          |
    |2017|7    |Trek Slash 8 27.5 - 2016                             |3          |
    |2017|7    |Trek X-Caliber 8 - 2017                              |3          |
    |2017|7    |Trek Session DH 27.5 Carbon Frameset - 2017          |3          |
    |2017|7    |Electra Moto 3i (20-inch) - Boy's - 2017             |3          |
    |2017|7    |Electra Townie Original 21D - 2016                   |3          |
    |2017|7    |Electra Amsterdam Original 3i - 2015/2017            |3          |
    |2017|7    |Sun Bicycles Streamway 7 - 2017                      |3          |
    |2017|7    |Electra Savannah 3i (20-inch) - Girl's - 2017        |3          |
    |2017|7    |Surly Straggler 650b - 2016                          |3          |
    |2017|7    |Sun Bicycles Spider 3i - 2017                        |3          |
    |2017|7    |Haro Flightline One ST - 2017                        |3          |
    |2017|7    |Trek Conduit+ - 2016                                 |3          |
    |2017|7    |Trek Domane S 6 - 2017                               |3          |
    |2017|7    |Sun Bicycles Cruz 3 - Women's - 2017                 |3          |
    |2017|7    |Surly Karate Monkey 27.5+ Frameset - 2017            |2          |
    |2017|7    |Haro SR 1.3 - 2017                                   |2          |
    |2017|7    |Sun Bicycles Brickell Tandem 7 - 2017                |2          |
    |2017|7    |Pure Cycles William 3-Speed - 2016                   |2          |
    |2017|7    |Sun Bicycles Biscayne Tandem 7 - 2017                |2          |
    |2017|7    |Trek Domane SL 6 - 2017                              |2          |
    |2017|7    |Electra Cruiser Lux 1 - 2017                         |2          |
    |2017|7    |Trek Precaliber 16 Boys - 2017                       |2          |
    |2017|7    |Electra Glam Punk 3i Ladies' - 2017                  |2          |
    |2017|7    |Electra Amsterdam Original 3i Ladies' - 2017         |2          |
    |2017|7    |Electra Straight 8 3i (20-inch) - Boy's - 2017       |2          |
    |2017|7    |Heller Shagamaw Frame - 2016                         |2          |
    |2017|7    |Electra Moto 1 - 2016                                |2          |
    |2017|7    |Haro Downtown 16 - 2017                              |2          |
    |2017|7    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |2          |
    |2017|7    |Sun Bicycles Cruz 7 - 2017                           |2          |
    |2017|7    |Trek Emonda S 5 - 2017                               |2          |
    |2017|7    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |2          |
    |2017|7    |Trek Girl's Kickster - 2017                          |2          |
    |2017|7    |Trek Boy's Kickster - 2015/2017                      |2          |
    |2017|7    |Trek Madone 9.2 - 2017                               |2          |
    |2017|7    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |2          |
    |2017|7    |Trek Domane SLR 6 Disc - 2017                        |2          |
    |2017|7    |Electra Cruiser 1 (24-Inch) - 2016                   |2          |
    |2017|7    |Haro Shredder 20 Girls - 2017                        |2          |
    |2017|7    |Trek Domane S 5 Disc - 2017                          |1          |
    |2017|7    |Pure Cycles Vine 8-Speed - 2016                      |1          |
    |2017|7    |Sun Bicycles Streamway - 2017                        |1          |
    |2017|7    |Ritchey Timberwolf Frameset - 2016                   |1          |
    |2017|7    |Surly Wednesday Frameset - 2017                      |1          |
    |2017|7    |Haro SR 1.1 - 2017                                   |1          |
    |2017|7    |Surly Steamroller - 2017                             |1          |
    |2017|7    |Trek Precaliber 16 Girls - 2017                      |1          |
    |2017|7    |Surly Ice Cream Truck Frameset - 2017                |1          |
    |2017|7    |Haro Flightline Two 26 Plus - 2017                   |1          |
    |2017|7    |Sun Bicycles Atlas X-Type - 2017                     |1          |
    |2017|7    |Trek Stache 5 - 2017                                 |1          |
    |2017|7    |Trek Precaliber 12 Girls - 2017                      |1          |
    |2017|7    |Surly Wednesday Frameset - 2016                      |1          |
    |2017|7    |Haro Shift R3 - 2017                                 |1          |
    |2017|7    |Sun Bicycles Brickell Tandem CB - 2017               |1          |
    |2017|7    |Trek Farley Alloy Frameset - 2017                    |1          |
    |2017|7    |Haro Shredder 20 - 2017                              |1          |
    |2017|8    |Surly Troll Frameset - 2017                          |8          |
    |2017|8    |Surly Ice Cream Truck Frameset - 2017                |8          |
    |2017|8    |Electra Townie Original 7D - 2017                    |7          |
    |2017|8    |Trek Precaliber 16 Boys - 2017                       |7          |
    |2017|8    |Electra Cruiser 1 (24-Inch) - 2016                   |7          |
    |2017|8    |Sun Bicycles ElectroLite - 2017                      |7          |
    |2017|8    |Haro SR 1.3 - 2017                                   |7          |
    |2017|8    |Electra Townie Original 21D - 2016                   |6          |
    |2017|8    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |6          |
    |2017|8    |Surly Wednesday Frameset - 2016                      |6          |
    |2017|8    |Sun Bicycles Biscayne Tandem 7 - 2017                |6          |
    |2017|8    |Surly Straggler - 2016                               |6          |
    |2017|8    |Electra Moto 1 - 2016                                |6          |
    |2017|8    |Electra Straight 8 3i (20-inch) - Boy's - 2017       |6          |
    |2017|8    |Haro SR 1.2 - 2017                                   |6          |
    |2017|8    |Trek Domane SL 6 - 2017                              |6          |
    |2017|8    |Electra Townie Original 7D EQ - 2016                 |6          |
    |2017|8    |Trek Powerfly 8 FS Plus - 2017                       |5          |
    |2017|8    |Electra Amsterdam Original 3i - 2015/2017            |5          |
    |2017|8    |Surly Karate Monkey 27.5+ Frameset - 2017            |5          |
    |2017|8    |Trek Conduit+ - 2016                                 |5          |
    |2017|8    |Electra Savannah 3i (20-inch) - Girl's - 2017        |5          |
    |2017|8    |Surly Straggler 650b - 2016                          |4          |
    |2017|8    |Sun Bicycles Lil Bolt Type-R - 2017                  |4          |
    |2017|8    |Trek X-Caliber 8 - 2017                              |4          |
    |2017|8    |Trek Emonda S 4 - 2017                               |4          |
    |2017|8    |Sun Bicycles Streamway 3 - 2017                      |4          |
    |2017|8    |Sun Bicycles Brickell Tandem 7 - 2017                |4          |
    |2017|8    |Haro Shredder Pro 20 - 2017                          |4          |
    |2017|8    |Sun Bicycles Drifter 7 - Women's - 2017              |4          |
    |2017|8    |Electra Cruiser Lux 1 - 2017                         |4          |
    |2017|8    |Surly Wednesday Frameset - 2017                      |4          |
    |2017|8    |Sun Bicycles Cruz 3 - 2017                           |4          |
    |2017|8    |Pure Cycles Vine 8-Speed - 2016                      |4          |
    |2017|8    |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |3          |
    |2017|8    |Sun Bicycles Cruz 7 - Women's - 2017                 |3          |
    |2017|8    |Electra Amsterdam Fashion 7i Ladies' - 2017          |3          |
    |2017|8    |Pure Cycles William 3-Speed - 2016                   |3          |
    |2017|8    |Trek Silque SLR 7 Women's - 2017                     |3          |
    |2017|8    |Sun Bicycles Biscayne Tandem CB - 2017               |3          |
    |2017|8    |Sun Bicycles Spider 3i - 2017                        |3          |
    |2017|8    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |3          |
    |2017|8    |Trek Farley Alloy Frameset - 2017                    |3          |
    |2017|8    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |3          |
    |2017|8    |Trek Precaliber 12 Boys - 2017                       |3          |
    |2017|8    |"Electra Girl's Hawaii 1 16"" - 2017"                |3          |
    |2017|8    |Trek Slash 8 27.5 - 2016                             |3          |
    |2017|8    |Trek Domane S 5 Disc - 2017                          |3          |
    |2017|8    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |3          |
    |2017|8    |Trek Boy's Kickster - 2015/2017                      |3          |
    |2017|8    |Trek Remedy 29 Carbon Frameset - 2016                |2          |
    |2017|8    |Surly Steamroller - 2017                             |2          |
    |2017|8    |Ritchey Timberwolf Frameset - 2016                   |2          |
    |2017|8    |Electra Glam Punk 3i Ladies' - 2017                  |2          |
    |2017|8    |Trek Domane S 6 - 2017                               |2          |
    |2017|8    |Trek Precaliber 16 Girls - 2017                      |2          |
    |2017|8    |Sun Bicycles Revolutions 24 - 2017                   |2          |
    |2017|8    |Sun Bicycles Streamway 7 - 2017                      |2          |
    |2017|8    |Heller Shagamaw Frame - 2016                         |2          |
    |2017|8    |Sun Bicycles Streamway - 2017                        |2          |
    |2017|8    |Haro Shift R3 - 2017                                 |2          |
    |2017|8    |Electra Townie 7D (20-inch) - Boys' - 2017           |2          |
    |2017|8    |Electra Townie Original 7D EQ - Women's - 2016       |2          |
    |2017|8    |Surly Wednesday - 2017                               |2          |
    |2017|8    |Surly Big Dummy Frameset - 2017                      |2          |
    |2017|8    |Surly Ice Cream Truck Frameset - 2016                |2          |
    |2017|8    |Electra Moto 3i (20-inch) - Boy's - 2017             |2          |
    |2017|8    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |2          |
    |2017|8    |Electra Townie Original 7D - 2015/2016               |2          |
    |2017|8    |Trek Girl's Kickster - 2017                          |1          |
    |2017|8    |Electra Amsterdam Original 3i Ladies' - 2017         |1          |
    |2017|8    |Sun Bicycles Revolutions 24 - Girl's - 2017          |1          |
    |2017|8    |Trek Boone Race Shop Limited - 2017                  |1          |
    |2017|8    |Trek Fuel EX 9.8 29 - 2017                           |1          |
    |2017|8    |Trek Silque SLR 8 Women's - 2017                     |1          |
    |2017|8    |Sun Bicycles Cruz 3 - Women's - 2017                 |1          |
    |2017|8    |Sun Bicycles Cruz 7 - 2017                           |1          |
    |2017|8    |Trek Session DH 27.5 Carbon Frameset - 2017          |1          |
    |2017|8    |Haro Flightline One ST - 2017                        |1          |
    |2017|8    |Trek Precaliber 24 (21-Speed) - Girls - 2017         |1          |
    |2017|8    |Trek Precaliber 12 Girls - 2017                      |1          |
    |2017|8    |Trek Domane SLR 6 Disc - 2017                        |1          |
    |2017|8    |Sun Bicycles Atlas X-Type - 2017                     |1          |
    |2017|8    |Sun Bicycles Drifter 7 - 2017                        |1          |
    |2017|8    |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |1          |
    |2017|8    |Haro Shredder 20 Girls - 2017                        |1          |
    |2017|9    |Sun Bicycles Atlas X-Type - 2017                     |6          |
    |2017|9    |Sun Bicycles Streamway 7 - 2017                      |6          |
    |2017|9    |Trek Precaliber 24 (21-Speed) - Girls - 2017         |6          |
    |2017|9    |Surly Straggler 650b - 2016                          |5          |
    |2017|9    |Trek Madone 9.2 - 2017                               |5          |
    |2017|9    |Surly Ice Cream Truck Frameset - 2017                |5          |
    |2017|9    |Sun Bicycles Cruz 7 - 2017                           |5          |
    |2017|9    |Surly Ice Cream Truck Frameset - 2016                |5          |
    |2017|9    |Sun Bicycles Cruz 3 - 2017                           |5          |
    |2017|9    |Electra Cruiser 1 (24-Inch) - 2016                   |5          |
    |2017|9    |Sun Bicycles Lil Bolt Type-R - 2017                  |4          |
    |2017|9    |Trek Powerfly 8 FS Plus - 2017                       |4          |
    |2017|9    |Trek Domane SLR 6 Disc - 2017                        |4          |
    |2017|9    |Trek Domane S 6 - 2017                               |4          |
    |2017|9    |Haro Shredder 20 Girls - 2017                        |4          |
    |2017|9    |Trek Conduit+ - 2016                                 |4          |
    |2017|9    |Trek Stache 5 - 2017                                 |4          |
    |2017|9    |Surly Troll Frameset - 2017                          |4          |
    |2017|9    |Haro SR 1.2 - 2017                                   |4          |
    |2017|9    |Sun Bicycles ElectroLite - 2017                      |4          |
    |2017|9    |Haro Shift R3 - 2017                                 |4          |
    |2017|9    |Sun Bicycles Cruz 7 - Women's - 2017                 |4          |
    |2017|9    |Electra Glam Punk 3i Ladies' - 2017                  |4          |
    |2017|9    |Trek Domane SL 6 - 2017                              |4          |
    |2017|9    |Haro Shredder Pro 20 - 2017                          |4          |
    |2017|9    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |3          |
    |2017|9    |Electra Townie Original 7D EQ - 2016                 |3          |
    |2017|9    |Haro SR 1.1 - 2017                                   |3          |
    |2017|9    |Pure Cycles Vine 8-Speed - 2016                      |3          |
    |2017|9    |Trek Slash 8 27.5 - 2016                             |3          |
    |2017|9    |Sun Bicycles Spider 3i - 2017                        |3          |
    |2017|9    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |3          |
    |2017|9    |Sun Bicycles Streamway 3 - 2017                      |3          |
    |2017|9    |Trek Silque SLR 8 Women's - 2017                     |3          |
    |2017|9    |Haro Flightline One ST - 2017                        |3          |
    |2017|9    |Electra Townie Original 21D - 2016                   |3          |
    |2017|9    |Trek Silque SLR 7 Women's - 2017                     |3          |
    |2017|9    |Sun Bicycles Streamway - 2017                        |3          |
    |2017|9    |Sun Bicycles Biscayne Tandem 7 - 2017                |3          |
    |2017|9    |Electra Savannah 3i (20-inch) - Girl's - 2017        |3          |
    |2017|9    |Electra Moto 3i (20-inch) - Boy's - 2017             |3          |
    |2017|9    |Trek Fuel EX 9.8 29 - 2017                           |2          |
    |2017|9    |Electra Amsterdam Fashion 7i Ladies' - 2017          |2          |
    |2017|9    |Electra Amsterdam Original 3i - 2015/2017            |2          |
    |2017|9    |Electra Townie 7D (20-inch) - Boys' - 2017           |2          |
    |2017|9    |Electra Townie Original 7D EQ - Women's - 2016       |2          |
    |2017|9    |Sun Bicycles Cruz 3 - Women's - 2017                 |2          |
    |2017|9    |Heller Shagamaw Frame - 2016                         |2          |
    |2017|9    |Electra Straight 8 3i (20-inch) - Boy's - 2017       |2          |
    |2017|9    |Trek Precaliber 12 Boys - 2017                       |2          |
    |2017|9    |Sun Bicycles Drifter 7 - 2017                        |2          |
    |2017|9    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |2          |
    |2017|9    |Electra Townie Original 7D - 2015/2016               |2          |
    |2017|9    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |2          |
    |2017|9    |Trek X-Caliber 8 - 2017                              |2          |
    |2017|9    |Trek Remedy 9.8 - 2017                               |2          |
    |2017|9    |Surly Wednesday Frameset - 2017                      |2          |
    |2017|9    |Electra Amsterdam Original 3i Ladies' - 2017         |2          |
    |2017|9    |Sun Bicycles Lil Kitt'n - 2017                       |2          |
    |2017|9    |Trek Precaliber 16 Girls - 2017                      |2          |
    |2017|9    |Haro Shredder 20 - 2017                              |2          |
    |2017|9    |Sun Bicycles Revolutions 24 - 2017                   |2          |
    |2017|9    |Trek Domane SL Disc Frameset - 2017                  |2          |
    |2017|9    |Surly Straggler - 2016                               |2          |
    |2017|9    |Trek Fuel EX 8 29 - 2016                             |2          |
    |2017|9    |Sun Bicycles Revolutions 24 - Girl's - 2017          |2          |
    |2017|9    |Trek Emonda S 4 - 2017                               |2          |
    |2017|9    |Trek Precaliber 16 Boys - 2017                       |2          |
    |2017|9    |Haro Flightline Two 26 Plus - 2017                   |2          |
    |2017|9    |Trek Farley Alloy Frameset - 2017                    |2          |
    |2017|9    |Sun Bicycles Drifter 7 - Women's - 2017              |2          |
    |2017|9    |Ritchey Timberwolf Frameset - 2016                   |2          |
    |2017|9    |Trek Precaliber 12 Girls - 2017                      |1          |
    |2017|9    |Electra Townie Original 7D - 2017                    |1          |
    |2017|9    |Haro Downtown 16 - 2017                              |1          |
    |2017|9    |Trek Fuel EX 5 27.5 Plus - 2017                      |1          |
    |2017|9    |Sun Bicycles Brickell Tandem CB - 2017               |1          |
    |2017|9    |Trek Boone 7 - 2017                                  |1          |
    |2017|9    |Trek Girl's Kickster - 2017                          |1          |
    |2017|9    |"Electra Girl's Hawaii 1 16"" - 2017"                |1          |
    |2017|9    |Surly Wednesday - 2017                               |1          |
    |2017|9    |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |1          |
    |2017|9    |Surly Steamroller - 2017                             |1          |
    |2017|9    |Trek Remedy 29 Carbon Frameset - 2016                |1          |
    |2017|9    |Surly Karate Monkey 27.5+ Frameset - 2017            |1          |
    |2017|9    |Surly Ogre Frameset - 2017                           |1          |
    |2017|10   |Sun Bicycles Cruz 3 - 2017                           |11         |
    |2017|10   |Electra Townie Original 7D - 2017                    |10         |
    |2017|10   |Haro Flightline One ST - 2017                        |8          |
    |2017|10   |Trek Fuel EX 9.8 29 - 2017                           |8          |
    |2017|10   |Electra Townie Original 21D - 2016                   |7          |
    |2017|10   |Sun Bicycles Streamway 3 - 2017                      |7          |
    |2017|10   |Electra Cruiser 1 (24-Inch) - 2016                   |7          |
    |2017|10   |Surly Ice Cream Truck Frameset - 2016                |7          |
    |2017|10   |Trek Boone 7 - 2017                                  |6          |
    |2017|10   |Electra Amsterdam Fashion 7i Ladies' - 2017          |6          |
    |2017|10   |Trek Conduit+ - 2016                                 |6          |
    |2017|10   |Electra Moto 1 - 2016                                |6          |
    |2017|10   |Sun Bicycles Atlas X-Type - 2017                     |6          |
    |2017|10   |Sun Bicycles Biscayne Tandem CB - 2017               |6          |
    |2017|10   |Surly Ogre Frameset - 2017                           |5          |
    |2017|10   |Trek Precaliber 12 Girls - 2017                      |5          |
    |2017|10   |Sun Bicycles Streamway 7 - 2017                      |5          |
    |2017|10   |Pure Cycles Vine 8-Speed - 2016                      |5          |
    |2017|10   |Trek X-Caliber 8 - 2017                              |5          |
    |2017|10   |Sun Bicycles Lil Bolt Type-R - 2017                  |5          |
    |2017|10   |Trek Boone Race Shop Limited - 2017                  |5          |
    |2017|10   |Sun Bicycles Cruz 3 - Women's - 2017                 |5          |
    |2017|10   |Pure Cycles William 3-Speed - 2016                   |5          |
    |2017|10   |Trek Precaliber 12 Boys - 2017                       |4          |
    |2017|10   |Trek Precaliber 24 (21-Speed) - Girls - 2017         |4          |
    |2017|10   |Sun Bicycles Brickell Tandem 7 - 2017                |4          |
    |2017|10   |Surly Straggler - 2016                               |4          |
    |2017|10   |Sun Bicycles Drifter 7 - 2017                        |4          |
    |2017|10   |Haro Shredder Pro 20 - 2017                          |4          |
    |2017|10   |Trek Remedy 9.8 - 2017                               |4          |
    |2017|10   |Sun Bicycles Lil Kitt'n - 2017                       |4          |
    |2017|10   |Trek Domane SL 6 - 2017                              |4          |
    |2017|10   |Sun Bicycles Revolutions 24 - 2017                   |4          |
    |2017|10   |Electra Townie Original 7D EQ - 2016                 |4          |
    |2017|10   |Trek Madone 9.2 - 2017                               |4          |
    |2017|10   |"Electra Girl's Hawaii 1 16"" - 2017"                |4          |
    |2017|10   |Haro Shredder 20 - 2017                              |4          |
    |2017|10   |Trek Fuel EX 5 27.5 Plus - 2017                      |4          |
    |2017|10   |Trek Slash 8 27.5 - 2016                             |3          |
    |2017|10   |Haro Shredder 20 Girls - 2017                        |3          |
    |2017|10   |Trek Precaliber 16 Boys - 2017                       |3          |
    |2017|10   |Haro Flightline Two 26 Plus - 2017                   |3          |
    |2017|10   |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |3          |
    |2017|10   |Sun Bicycles Brickell Tandem CB - 2017               |3          |
    |2017|10   |Electra Savannah 3i (20-inch) - Girl's - 2017        |3          |
    |2017|10   |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |3          |
    |2017|10   |Haro SR 1.3 - 2017                                   |3          |
    |2017|10   |Sun Bicycles Spider 3i - 2017                        |3          |
    |2017|10   |Electra Glam Punk 3i Ladies' - 2017                  |3          |
    |2017|10   |Haro Shift R3 - 2017                                 |3          |
    |2017|10   |Trek Girl's Kickster - 2017                          |3          |
    |2017|10   |Electra Cruiser Lux 1 - 2017                         |3          |
    |2017|10   |Surly Troll Frameset - 2017                          |2          |
    |2017|10   |Trek Emonda S 4 - 2017                               |2          |
    |2017|10   |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |2          |
    |2017|10   |Haro Downtown 16 - 2017                              |2          |
    |2017|10   |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |2          |
    |2017|10   |Sun Bicycles Cruz 7 - Women's - 2017                 |2          |
    |2017|10   |Surly Wednesday Frameset - 2017                      |2          |
    |2017|10   |Trek Stache 5 - 2017                                 |2          |
    |2017|10   |Haro SR 1.1 - 2017                                   |2          |
    |2017|10   |Trek Boy's Kickster - 2015/2017                      |2          |
    |2017|10   |Sun Bicycles Cruz 7 - 2017                           |2          |
    |2017|10   |Surly Wednesday - 2017                               |2          |
    |2017|10   |Trek Silque SLR 8 Women's - 2017                     |2          |
    |2017|10   |Sun Bicycles Streamway - 2017                        |2          |
    |2017|10   |Sun Bicycles Revolutions 24 - Girl's - 2017          |2          |
    |2017|10   |Surly Big Dummy Frameset - 2017                      |2          |
    |2017|10   |Trek Farley Alloy Frameset - 2017                    |2          |
    |2017|10   |Electra Townie Original 7D EQ - Women's - 2016       |2          |
    |2017|10   |Trek Domane S 6 - 2017                               |2          |
    |2017|10   |Surly Wednesday Frameset - 2016                      |1          |
    |2017|10   |Trek Fuel EX 9.8 27.5 Plus - 2017                    |1          |
    |2017|10   |Surly Steamroller - 2017                             |1          |
    |2017|10   |Electra Straight 8 3i (20-inch) - Boy's - 2017       |1          |
    |2017|10   |Trek Fuel EX 8 29 - 2016                             |1          |
    |2017|10   |Electra Amsterdam Original 3i Ladies' - 2017         |1          |
    |2017|10   |Trek Silque SLR 7 Women's - 2017                     |1          |
    |2017|10   |Trek Remedy 29 Carbon Frameset - 2016                |1          |
    |2017|10   |Trek Domane S 5 Disc - 2017                          |1          |
    |2017|10   |Trek Domane SL Disc Frameset - 2017                  |1          |
    |2017|11   |Trek Precaliber 12 Boys - 2017                       |8          |
    |2017|11   |Trek Farley Alloy Frameset - 2017                    |7          |
    |2017|11   |Electra Glam Punk 3i Ladies' - 2017                  |7          |
    |2017|11   |Surly Ice Cream Truck Frameset - 2016                |6          |
    |2017|11   |Trek Powerfly 8 FS Plus - 2017                       |6          |
    |2017|11   |Electra Moto 1 - 2016                                |6          |
    |2017|11   |Sun Bicycles Atlas X-Type - 2017                     |6          |
    |2017|11   |Sun Bicycles Biscayne Tandem CB - 2017               |6          |
    |2017|11   |Electra Townie Original 7D EQ - 2016                 |6          |
    |2017|11   |"Electra Girl's Hawaii 1 16"" - 2017"                |5          |
    |2017|11   |Trek Fuel EX 8 29 - 2016                             |5          |
    |2017|11   |Trek Domane S 6 - 2017                               |5          |
    |2017|11   |Trek Fuel EX 5 27.5 Plus - 2017                      |5          |
    |2017|11   |Haro SR 1.2 - 2017                                   |5          |
    |2017|11   |Haro Shredder Pro 20 - 2017                          |5          |
    |2017|11   |Electra Townie Original 7D EQ - Women's - 2016       |5          |
    |2017|11   |Electra Townie Original 7D - 2015/2016               |5          |
    |2017|11   |Trek Fuel EX 9.8 27.5 Plus - 2017                    |4          |
    |2017|11   |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |4          |
    |2017|11   |Haro SR 1.1 - 2017                                   |4          |
    |2017|11   |Trek Domane S 5 Disc - 2017                          |4          |
    |2017|11   |Haro Flightline Two 26 Plus - 2017                   |4          |
    |2017|11   |Trek Slash 8 27.5 - 2016                             |3          |
    |2017|11   |Haro Flightline One ST - 2017                        |3          |
    |2017|11   |Trek Domane SLR 6 Disc - 2017                        |3          |
    |2017|11   |Trek Precaliber 24 (21-Speed) - Girls - 2017         |3          |
    |2017|11   |Trek Precaliber 12 Girls - 2017                      |3          |
    |2017|11   |Trek Boone 7 - 2017                                  |3          |
    |2017|11   |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |3          |
    |2017|11   |Trek Precaliber 16 Girls - 2017                      |3          |
    |2017|11   |Electra Townie Original 21D - 2016                   |3          |
    |2017|11   |Trek Madone 9.2 - 2017                               |3          |
    |2017|11   |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |3          |
    |2017|11   |Sun Bicycles Revolutions 24 - 2017                   |3          |
    |2017|11   |Trek Fuel EX 9.8 29 - 2017                           |3          |
    |2017|11   |Surly Ice Cream Truck Frameset - 2017                |3          |
    |2017|11   |Pure Cycles William 3-Speed - 2016                   |3          |
    |2017|11   |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |3          |
    |2017|11   |Trek Emonda S 5 - 2017                               |3          |
    |2017|11   |Surly Wednesday Frameset - 2017                      |3          |
    |2017|11   |Electra Townie 7D (20-inch) - Boys' - 2017           |3          |
    |2017|11   |Surly Troll Frameset - 2017                          |3          |
    |2017|11   |Haro Shift R3 - 2017                                 |3          |
    |2017|11   |Sun Bicycles ElectroLite - 2017                      |2          |
    |2017|11   |Surly Big Dummy Frameset - 2017                      |2          |
    |2017|11   |Trek Session DH 27.5 Carbon Frameset - 2017          |2          |
    |2017|11   |Surly Karate Monkey 27.5+ Frameset - 2017            |2          |
    |2017|11   |Trek Silque SLR 7 Women's - 2017                     |2          |
    |2017|11   |Sun Bicycles Cruz 7 - 2017                           |2          |
    |2017|11   |Haro Shredder 20 - 2017                              |2          |
    |2017|11   |Sun Bicycles Biscayne Tandem 7 - 2017                |2          |
    |2017|11   |Trek Remedy 29 Carbon Frameset - 2016                |2          |
    |2017|11   |Trek Silque SLR 8 Women's - 2017                     |2          |
    |2017|11   |Pure Cycles Vine 8-Speed - 2016                      |2          |
    |2017|11   |Sun Bicycles Cruz 3 - 2017                           |2          |
    |2017|11   |Electra Cruiser 1 (24-Inch) - 2016                   |2          |
    |2017|11   |Surly Steamroller - 2017                             |2          |
    |2017|11   |Pure Cycles Western 3-Speed - Women's - 2015/2016    |2          |
    |2017|11   |Electra Amsterdam Fashion 7i Ladies' - 2017          |2          |
    |2017|11   |Trek X-Caliber 8 - 2017                              |2          |
    |2017|11   |Surly Straggler 650b - 2016                          |2          |
    |2017|11   |Sun Bicycles Drifter 7 - 2017                        |2          |
    |2017|11   |Electra Amsterdam Original 3i - 2015/2017            |2          |
    |2017|11   |Electra Cruiser Lux Fat Tire 1 Ladies - 2017         |2          |
    |2017|11   |Sun Bicycles Cruz 3 - Women's - 2017                 |2          |
    |2017|11   |Electra Savannah 3i (20-inch) - Girl's - 2017        |1          |
    |2017|11   |Electra Amsterdam Original 3i Ladies' - 2017         |1          |
    |2017|11   |Haro SR 1.3 - 2017                                   |1          |
    |2017|11   |Sun Bicycles Brickell Tandem CB - 2017               |1          |
    |2017|11   |Trek Domane SL Disc Frameset - 2017                  |1          |
    |2017|11   |Trek Girl's Kickster - 2017                          |1          |
    |2017|11   |Surly Straggler - 2016                               |1          |
    |2017|11   |Sun Bicycles Streamway 3 - 2017                      |1          |
    |2017|11   |Sun Bicycles Drifter 7 - Women's - 2017              |1          |
    |2017|11   |Sun Bicycles Lil Kitt'n - 2017                       |1          |
    |2017|11   |Sun Bicycles Brickell Tandem 7 - 2017                |1          |
    |2017|11   |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |1          |
    |2017|11   |Electra Cruiser Lux 1 - 2017                         |1          |
    |2017|11   |Sun Bicycles Streamway - 2017                        |1          |
    |2017|11   |Surly Ogre Frameset - 2017                           |1          |
    |2017|11   |Trek Emonda S 4 - 2017                               |1          |
    |2017|12   |Sun Bicycles Cruz 7 - 2017                           |8          |
    |2017|12   |Sun Bicycles Streamway - 2017                        |7          |
    |2017|12   |Surly Straggler 650b - 2016                          |7          |
    |2017|12   |Trek Conduit+ - 2016                                 |7          |
    |2017|12   |Electra Cruiser Lux Fat Tire 1 Ladies - 2017         |6          |
    |2017|12   |Trek Boone 7 - 2017                                  |6          |
    |2017|12   |Surly Big Dummy Frameset - 2017                      |6          |
    |2017|12   |Haro SR 1.2 - 2017                                   |5          |
    |2017|12   |Surly Steamroller - 2017                             |5          |
    |2017|12   |Trek Domane SLR 6 Disc - 2017                        |5          |
    |2017|12   |Sun Bicycles Streamway 3 - 2017                      |5          |
    |2017|12   |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |5          |
    |2017|12   |Electra Amsterdam Fashion 7i Ladies' - 2017          |5          |
    |2017|12   |Electra Townie 7D (20-inch) - Boys' - 2017           |4          |
    |2017|12   |Electra Townie Original 7D EQ - 2016                 |4          |
    |2017|12   |"Electra Girl's Hawaii 1 16"" - 2017"                |4          |
    |2017|12   |Haro Shredder 20 Girls - 2017                        |4          |
    |2017|12   |Trek Madone 9.2 - 2017                               |4          |
    |2017|12   |Sun Bicycles Biscayne Tandem 7 - 2017                |4          |
    |2017|12   |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |4          |
    |2017|12   |Electra Townie Original 7D EQ - Women's - 2016       |4          |
    |2017|12   |Electra Moto 3i (20-inch) - Boy's - 2017             |3          |
    |2017|12   |Electra Townie Original 7D - 2017                    |3          |
    |2017|12   |Trek Precaliber 12 Boys - 2017                       |3          |
    |2017|12   |Haro Shredder 20 - 2017                              |3          |
    |2017|12   |Pure Cycles Vine 8-Speed - 2016                      |3          |
    |2017|12   |Trek Precaliber 24 (21-Speed) - Girls - 2017         |3          |
    |2017|12   |Electra Amsterdam Original 3i Ladies' - 2017         |3          |
    |2017|12   |Electra Cruiser Lux 1 - 2017                         |3          |
    |2017|12   |Trek Fuel EX 8 29 - 2016                             |3          |
    |2017|12   |Trek Powerfly 8 FS Plus - 2017                       |3          |
    |2017|12   |Trek Boone Race Shop Limited - 2017                  |3          |
    |2017|12   |Electra Townie Original 21D - 2016                   |3          |
    |2017|12   |Trek Farley Alloy Frameset - 2017                    |3          |
    |2017|12   |Haro Flightline Two 26 Plus - 2017                   |3          |
    |2017|12   |Trek Remedy 29 Carbon Frameset - 2016                |2          |
    |2017|12   |Sun Bicycles Revolutions 24 - Girl's - 2017          |2          |
    |2017|12   |Trek Silque SLR 7 Women's - 2017                     |2          |
    |2017|12   |Trek Precaliber 12 Girls - 2017                      |2          |
    |2017|12   |Surly Troll Frameset - 2017                          |2          |
    |2017|12   |Surly Karate Monkey 27.5+ Frameset - 2017            |2          |
    |2017|12   |Trek Domane SL 6 - 2017                              |2          |
    |2017|12   |Trek Fuel EX 5 27.5 Plus - 2017                      |2          |
    |2017|12   |Sun Bicycles Brickell Tandem 7 - 2017                |2          |
    |2017|12   |Trek Silque SLR 8 Women's - 2017                     |2          |
    |2017|12   |Electra Glam Punk 3i Ladies' - 2017                  |2          |
    |2017|12   |Trek Domane SL Disc Frameset - 2017                  |2          |
    |2017|12   |Sun Bicycles Drifter 7 - 2017                        |2          |
    |2017|12   |Sun Bicycles Drifter 7 - Women's - 2017              |2          |
    |2017|12   |Electra Townie Original 7D - 2015/2016               |2          |
    |2017|12   |Sun Bicycles Cruz 7 - Women's - 2017                 |2          |
    |2017|12   |Surly Ogre Frameset - 2017                           |2          |
    |2017|12   |Heller Shagamaw Frame - 2016                         |2          |
    |2017|12   |Trek Precaliber 16 Girls - 2017                      |2          |
    |2017|12   |Sun Bicycles Streamway 7 - 2017                      |2          |
    |2017|12   |Sun Bicycles Biscayne Tandem CB - 2017               |2          |
    |2017|12   |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |2          |
    |2017|12   |Trek Domane S 5 Disc - 2017                          |2          |
    |2017|12   |Electra Amsterdam Original 3i - 2015/2017            |2          |
    |2017|12   |Ritchey Timberwolf Frameset - 2016                   |2          |
    |2017|12   |Surly Wednesday Frameset - 2017                      |1          |
    |2017|12   |Trek Precaliber 16 Boys - 2017                       |1          |
    |2017|12   |Haro Shredder Pro 20 - 2017                          |1          |
    |2017|12   |Sun Bicycles Brickell Tandem CB - 2017               |1          |
    |2017|12   |Trek Session DH 27.5 Carbon Frameset - 2017          |1          |
    |2017|12   |Trek Fuel EX 9.8 27.5 Plus - 2017                    |1          |
    |2017|12   |Haro SR 1.1 - 2017                                   |1          |
    |2017|12   |Surly Ice Cream Truck Frameset - 2016                |1          |
    |2017|12   |Haro Downtown 16 - 2017                              |1          |
    |2017|12   |Haro Shift R3 - 2017                                 |1          |
    |2017|12   |Trek Slash 8 27.5 - 2016                             |1          |
    |2017|12   |Surly Wednesday Frameset - 2016                      |1          |
    |2017|12   |Trek Emonda S 5 - 2017                               |1          |
    |2017|12   |Sun Bicycles Lil Bolt Type-R - 2017                  |1          |
    |2017|12   |Sun Bicycles Cruz 3 - Women's - 2017                 |1          |
    |2018|1    |Electra Cruiser 1 (24-Inch) - 2016                   |6          |
    |2018|1    |Sun Bicycles Revolutions 24 - 2017                   |6          |
    |2018|1    |Trek Girl's Kickster - 2017                          |4          |
    |2018|1    |Trek Fuel EX 5 Plus - 2018                           |4          |
    |2018|1    |Trek Super Commuter+ 8S - 2018                       |4          |
    |2018|1    |Electra Cyclosaurus 1 (16-inch) - Boy's - 2018       |4          |
    |2018|1    |Trek Domane ALR 5 Disc - 2018                        |4          |
    |2018|1    |Electra Townie Go! 8i Ladies' - 2018                 |4          |
    |2018|1    |Electra Heartchya 1 (20-inch) - Girl's - 2018        |4          |
    |2018|1    |Trek Domane ALR 5 Gravel - 2018                      |3          |
    |2018|1    |Electra Koa 3i Ladies' - 2018                        |3          |
    |2018|1    |Trek Madone 9.2 - 2017                               |3          |
    |2018|1    |Electra Townie Balloon 8D EQ - 2016/2017/2018        |3          |
    |2018|1    |Trek Powerfly 7 FS - 2018                            |3          |
    |2018|1    |Sun Bicycles Cruz 7 - 2017                           |3          |
    |2018|1    |Trek Remedy 27.5 C Frameset - 2018                   |3          |
    |2018|1    |Electra Townie Original 21D - 2016                   |3          |
    |2018|1    |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |3          |
    |2018|1    |"Electra Under-The-Sea 1 16"" - 2018"                |3          |
    |2018|1    |Electra Amsterdam Fashion 3i Ladies' - 2017/2018     |3          |
    |2018|1    |Trek Stache Carbon Frameset - 2018                   |3          |
    |2018|1    |Trek Domane SL 6 Disc - 2018                         |3          |
    |2018|1    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |2          |
    |2018|1    |Trek Domane SL 5 - 2018                              |2          |
    |2018|1    |Trek Emonda S 4 - 2017                               |2          |
    |2018|1    |Sun Bicycles Biscayne Tandem CB - 2017               |2          |
    |2018|1    |Electra Daydreamer 3i Ladies' - 2018                 |2          |
    |2018|1    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |2          |
    |2018|1    |Surly ECR 27.5 - 2018                                |2          |
    |2018|1    |Haro Shredder 20 Girls - 2017                        |2          |
    |2018|1    |Trek Domane ALR 4 Disc Women's - 2018                |2          |
    |2018|1    |Trek Super Commuter+ 7 - 2018                        |2          |
    |2018|1    |Trek Fuel EX 7 29 - 2018                             |2          |
    |2018|1    |Trek Precaliber 12 Boy's - 2018                      |2          |
    |2018|1    |Sun Bicycles Streamway 3 - 2017                      |2          |
    |2018|1    |Strider Classic 12 Balance Bike - 2018               |2          |
    |2018|1    |Electra Townie Commute 8D - 2018                     |2          |
    |2018|1    |Electra Cruiser Lux 1 Ladies' - 2018                 |2          |
    |2018|1    |Surly Straggler - 2018                               |2          |
    |2018|1    |Electra Townie Original 1 Ladies' - 2018             |2          |
    |2018|1    |Trek Dual Sport+ - 2018                              |2          |
    |2018|1    |Surly Straggler 650b - 2018                          |2          |
    |2018|1    |Electra Townie Original 7D - 2017                    |2          |
    |2018|1    |Trek X-Caliber 8 - 2017                              |2          |
    |2018|1    |Electra Cruiser 1 - 2016/2017/2018                   |2          |
    |2018|1    |Trek Emonda SLR 8 - 2018                             |2          |
    |2018|1    |Trek Domane AL 2 Women's - 2018                      |2          |
    |2018|1    |Electra Townie Original 21D EQ - 2017/2018           |2          |
    |2018|1    |Sun Bicycles Lil Bolt Type-R - 2017                  |2          |
    |2018|1    |Trek Remedy 29 Carbon Frameset - 2016                |2          |
    |2018|1    |Surly Steamroller - 2017                             |2          |
    |2018|1    |Surly Big Dummy Frameset - 2017                      |2          |
    |2018|1    |Trek Fuel EX 9.8 29 - 2017                           |2          |
    |2018|1    |Sun Bicycles ElectroLite - 2017                      |2          |
    |2018|1    |Trek Precaliber 24 (7-Speed) - Boys - 2018           |2          |
    |2018|1    |Electra Cruiser Lux 3i - 2018                        |2          |
    |2018|1    |Trek Domane SL 7 Women's - 2018                      |2          |
    |2018|1    |Trek Marlin 5 - 2018                                 |2          |
    |2018|1    |Electra Queen of Hearts 3i - 2018                    |2          |
    |2018|1    |Surly Wednesday Frameset - 2016                      |2          |
    |2018|1    |Trek Precaliber 16 Boys - 2017                       |2          |
    |2018|1    |Electra Townie Go! 8i - 2017/2018                    |2          |
    |2018|1    |Electra Townie Balloon 3i EQ Ladies' - 2018          |2          |
    |2018|1    |Haro Shredder 20 - 2017                              |2          |
    |2018|1    |Trek Crockett 5 Disc - 2018                          |2          |
    |2018|1    |Heller Bloodhound Trail - 2018                       |2          |
    |2018|1    |Trek Domane AL 3 - 2018                              |2          |
    |2018|1    |Sun Bicycles Lil Kitt'n - 2017                       |2          |
    |2018|1    |Electra Townie Commute Go! Ladies' - 2018            |2          |
    |2018|1    |Trek Domane ALR Disc Frameset - 2018                 |2          |
    |2018|1    |Electra Glam Punk 3i Ladies' - 2017                  |2          |
    |2018|1    |Trek Domane SLR 6 Disc - 2017                        |2          |
    |2018|1    |Electra Townie Original 21D EQ Ladies' - 2018        |2          |
    |2018|1    |Ritchey Timberwolf Frameset - 2016                   |2          |
    |2018|1    |Trek Emonda SLR 6 - 2018                             |2          |
    |2018|1    |Electra Relic 3i - 2018                              |2          |
    |2018|1    |Surly Pack Rat - 2018                                |2          |
    |2018|1    |Trek X-Caliber 7 - 2018                              |2          |
    |2018|1    |Trek Fuel EX 8 29 - 2016                             |2          |
    |2018|1    |Trek Domane SLR 9 Disc - 2018                        |2          |
    |2018|1    |Electra White Water 3i - 2018                        |1          |
    |2018|1    |Trek Conduit+ - 2018                                 |1          |
    |2018|1    |Trek Domane SLR Frameset - 2018                      |1          |
    |2018|1    |"Electra Superbolt 3i 20"" - 2018"                   |1          |
    |2018|1    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |1          |
    |2018|1    |Electra Amsterdam Original 3i Ladies' - 2017         |1          |
    |2018|1    |Surly Krampus - 2018                                 |1          |
    |2018|1    |Trek Domane SL 5 Disc - 2018                         |1          |
    |2018|1    |Electra Townie Balloon 3i EQ - 2017/2018             |1          |
    |2018|1    |Trek Procaliber 6 - 2018                             |1          |
    |2018|1    |Trek Domane ALR 4 Disc - 2018                        |1          |
    |2018|1    |Trek Domane SL Frameset Women's - 2018               |1          |
    |2018|1    |Trek CrossRip+ - 2018                                |1          |
    |2018|1    |Sun Bicycles Drifter 7 - 2017                        |1          |
    |2018|1    |Trek Procal AL Frameset - 2018                       |1          |
    |2018|1    |Electra Cruiser Lux 7D Ladies' - 2018                |1          |
    |2018|1    |Surly Ogre Frameset - 2017                           |1          |
    |2018|1    |Trek Domane ALR 3 - 2018                             |1          |
    |2018|1    |Trek Remedy 7 27.5 - 2018                            |1          |
    |2018|1    |Trek Crockett 7 Disc - 2018                          |1          |
    |2018|1    |Trek Domane S 6 - 2017                               |1          |
    |2018|1    |Trek Domane SL 8 Disc - 2018                         |1          |
    |2018|1    |Trek Emonda ALR 6 - 2018                             |1          |
    |2018|1    |Electra Water Lily 1 (16-inch) - Girl's - 2018       |1          |
    |2018|1    |Surly Wednesday - 2017                               |1          |
    |2018|1    |Electra Townie Original 7D EQ - 2018                 |1          |
    |2018|1    |Electra Morningstar 3i Ladies' - 2018                |1          |
    |2018|1    |"Electra Treasure 3i 20"" - 2018"                    |1          |
    |2018|1    |Trek Boone Race Shop Limited - 2017                  |1          |
    |2018|1    |Trek Powerfly 8 FS Plus - 2017                       |1          |
    |2018|1    |Electra Townie Original 21D Ladies' - 2018           |1          |
    |2018|1    |Surly Karate Monkey 27.5+ Frameset - 2017            |1          |
    |2018|1    |Trek Boone 5 Disc - 2018                             |1          |
    |2018|1    |Trek Powerfly 5 - 2018                               |1          |
    |2018|1    |Trek Fuel EX 8 29 - 2018                             |1          |
    |2018|1    |Trek Slash 8 27.5 - 2016                             |1          |
    |2018|1    |Trek Domane AL 3 Women's - 2018                      |1          |
    |2018|1    |Trek Boone 7 Disc - 2018                             |1          |
    |2018|1    |Trek Precaliber 16 Boy's - 2018                      |1          |
    |2018|1    |Trek Fuel EX 5 27.5 Plus - 2017                      |1          |
    |2018|1    |Trek Emonda SL 7 - 2018                              |1          |
    |2018|1    |Trek Domane SLR 8 Disc - 2018                        |1          |
    |2018|1    |Trek Procaliber Frameset - 2018                      |1          |
    |2018|1    |Electra Cruiser 7D Tall - 2016/2018                  |1          |
    |2018|1    |Electra Cruiser Lux Fat Tire 7D - 2018               |1          |
    |2018|1    |Trek Domane SL 5 Disc Women's - 2018                 |1          |
    |2018|2    |Electra Townie Balloon 7i EQ Ladies' - 2017/2018     |4          |
    |2018|2    |Electra Townie Original 21D - 2016                   |4          |
    |2018|2    |Trek Domane SL 6 - 2018                              |4          |
    |2018|2    |Surly Pack Rat Frameset - 2018                       |4          |
    |2018|2    |Trek XM700+ Lowstep - 2018                           |4          |
    |2018|2    |Trek Fuel EX 8 29 XT - 2018                          |4          |
    |2018|2    |Electra Cruiser Lux 1 Ladies' - 2018                 |3          |
    |2018|2    |Surly Karate Monkey 27.5+ Frameset - 2017            |3          |
    |2018|2    |Trek Powerfly 7 FS - 2018                            |3          |
    |2018|2    |Trek Emonda S 4 - 2017                               |3          |
    |2018|2    |Trek 820 - 2018                                      |3          |
    |2018|2    |Haro Downtown 16 - 2017                              |2          |
    |2018|2    |Electra Townie Commute 27D Ladies - 2018             |2          |
    |2018|2    |Trek Emonda SL 6 Disc - 2018                         |2          |
    |2018|2    |Trek Domane SL 7 Women's - 2018                      |2          |
    |2018|2    |Trek Domane AL 3 Women's - 2018                      |2          |
    |2018|2    |Trek X-Caliber Frameset - 2018                       |2          |
    |2018|2    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |2          |
    |2018|2    |Trek Stache 5 - 2017                                 |2          |
    |2018|2    |Electra Tiger Shark 3i - 2018                        |2          |
    |2018|2    |Electra Moto 3i (20-inch) - Boy's - 2017             |2          |
    |2018|2    |Sun Bicycles Biscayne Tandem CB - 2017               |2          |
    |2018|2    |Haro Shift R3 - 2017                                 |2          |
    |2018|2    |Trek Precaliber 12 Boys - 2017                       |2          |
    |2018|2    |Surly Straggler - 2018                               |2          |
    |2018|2    |Surly Big Dummy Frameset - 2017                      |2          |
    |2018|2    |Heller Shagamaw GX1 - 2018                           |2          |
    |2018|2    |Trek 1120 - 2018                                     |2          |
    |2018|2    |Electra Townie Balloon 7i EQ - 2018                  |2          |
    |2018|2    |Surly Troll Frameset - 2017                          |2          |
    |2018|2    |Strider Classic 12 Balance Bike - 2018               |2          |
    |2018|2    |Electra Amsterdam Fashion 7i Ladies' - 2017          |2          |
    |2018|2    |Electra Straight 8 3i - 2018                         |2          |
    |2018|2    |Trek Super Commuter+ 8S - 2018                       |2          |
    |2018|2    |Trek Remedy 9.8 - 2017                               |2          |
    |2018|2    |Electra Townie Balloon 3i EQ - 2017/2018             |2          |
    |2018|2    |Trek Domane ALR 5 Gravel - 2018                      |2          |
    |2018|2    |Trek Lift+ - 2018                                    |2          |
    |2018|2    |Trek Kickster - 2018                                 |2          |
    |2018|2    |Electra Cruiser Lux 3i - 2018                        |2          |
    |2018|2    |Trek X-Caliber 7 - 2018                              |2          |
    |2018|2    |Trek Precaliber 20 Boy's - 2018                      |2          |
    |2018|2    |Sun Bicycles Lil Bolt Type-R - 2017                  |2          |
    |2018|2    |Electra Townie Original 21D EQ - 2017/2018           |2          |
    |2018|2    |Trek Lift+ Lowstep - 2018                            |2          |
    |2018|2    |Sun Bicycles Cruz 7 - 2017                           |2          |
    |2018|2    |"Electra Treasure 1 20"" - 2018"                     |2          |
    |2018|2    |Electra Cruiser Lux 1 - 2016/2018                    |2          |
    |2018|2    |Ritchey Timberwolf Frameset - 2016                   |2          |
    |2018|2    |Haro Shredder 20 Girls - 2017                        |2          |
    |2018|2    |Trek Precaliber 16 Boys - 2017                       |1          |
    |2018|2    |Trek CrossRip 2 - 2018                               |1          |
    |2018|2    |Trek Domane SL 5 - 2018                              |1          |
    |2018|2    |Trek Remedy 7 27.5 - 2018                            |1          |
    |2018|2    |Electra Cruiser 1 Tall - 2016/2018                   |1          |
    |2018|2    |Sun Bicycles Drifter 7 - 2017                        |1          |
    |2018|2    |Trek Farley Alloy Frameset - 2017                    |1          |
    |2018|2    |Electra Savannah 3i (20-inch) - Girl's - 2017        |1          |
    |2018|2    |Trek Super Commuter+ 7 - 2018                        |1          |
    |2018|2    |Trek MT 201 - 2018                                   |1          |
    |2018|2    |Trek Marlin 6 - 2018                                 |1          |
    |2018|2    |Electra Moto 1 - 2016                                |1          |
    |2018|2    |Electra Townie Balloon 3i EQ Ladies' - 2018          |1          |
    |2018|2    |Heller Shagamaw Frame - 2016                         |1          |
    |2018|2    |Electra Delivery 3i - 2016/2017/2018                 |1          |
    |2018|2    |Trek Emonda S 5 - 2017                               |1          |
    |2018|2    |Trek Domane AL 2 Women's - 2018                      |1          |
    |2018|2    |Trek Stache Carbon Frameset - 2018                   |1          |
    |2018|2    |Trek Marlin 5 - 2018                                 |1          |
    |2018|2    |Sun Bicycles Streamway - 2017                        |1          |
    |2018|2    |Trek Precaliber 20 6-speed Girl's - 2018             |1          |
    |2018|2    |Trek Precaliber 24 21-speed Boy's - 2018             |1          |
    |2018|2    |Electra Townie Balloon 8D EQ - 2016/2017/2018        |1          |
    |2018|2    |Sun Bicycles Cruz 3 - 2017                           |1          |
    |2018|2    |Electra Cruiser 1 Ladies' - 2018                     |1          |
    |2018|2    |Trek Precaliber 20 Girl's - 2018                     |1          |
    |2018|2    |Electra Cruiser Lux 7D Ladies' - 2018                |1          |
    |2018|2    |Trek Domane ALR 5 Disc - 2018                        |1          |
    |2018|2    |Electra Sweet Ride 3i (20-inch) - Girls' - 2018      |1          |
    |2018|2    |Trek Superfly 20 - 2018                              |1          |
    |2018|2    |Electra Amsterdam Original 3i Ladies' - 2017         |1          |
    |2018|2    |Electra Townie Commute Go! - 2018                    |1          |
    |2018|2    |Sun Bicycles Spider 3i - 2017                        |1          |
    |2018|2    |Trek Emonda SL 7 - 2018                              |1          |
    |2018|2    |Electra White Water 3i - 2018                        |1          |
    |2018|2    |Trek X-Caliber 8 - 2018                              |1          |
    |2018|2    |Surly Ogre Frameset - 2017                           |1          |
    |2018|2    |Trek Precaliber 16 Girls - 2017                      |1          |
    |2018|2    |Electra Cruiser 7D (24-Inch) Ladies' - 2016/2018     |1          |
    |2018|2    |Strider Strider 20 Sport - 2018                      |1          |
    |2018|2    |Trek Slash 8 27.5 - 2016                             |1          |
    |2018|2    |Haro Shredder 20 - 2017                              |1          |
    |2018|2    |Electra Cruiser Lux 7D - 2018                        |1          |
    |2018|3    |Sun Bicycles Cruz 7 - Women's - 2017                 |7          |
    |2018|3    |Trek Farley Carbon Frameset - 2018                   |6          |
    |2018|3    |Electra Townie Commute 8D Ladies' - 2018             |6          |
    |2018|3    |Trek XM700+ - 2018                                   |5          |
    |2018|3    |Electra Townie Commute Go! Ladies' - 2018            |5          |
    |2018|3    |Electra Cruiser 7D (24-Inch) Ladies' - 2016/2018     |5          |
    |2018|3    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |4          |
    |2018|3    |Electra Moto 3i - 2018                               |4          |
    |2018|3    |Trek Stache 5 - 2018                                 |4          |
    |2018|3    |Electra Townie Balloon 3i EQ Ladies' - 2018          |4          |
    |2018|3    |Electra Amsterdam Original 3i - 2015/2017            |4          |
    |2018|3    |Electra Townie 7D (20-inch) - Boys' - 2017           |4          |
    |2018|3    |Electra Koa 3i Ladies' - 2018                        |4          |
    |2018|3    |Trek Domane AL 3 Women's - 2018                      |4          |
    |2018|3    |Electra Daydreamer 3i Ladies' - 2018                 |4          |
    |2018|3    |Trek Kickster - 2018                                 |3          |
    |2018|3    |Surly ECR - 2018                                     |3          |
    |2018|3    |Strider Sport 16 - 2018                              |3          |
    |2018|3    |Trek Marlin 5 - 2018                                 |3          |
    |2018|3    |Electra Townie Original 7D - 2017                    |3          |
    |2018|3    |Electra Townie Balloon 7i EQ - 2018                  |3          |
    |2018|3    |Electra Townie Balloon 8D EQ Ladies' - 2016/2017/2018|3          |
    |2018|3    |Electra Townie Commute Go! - 2018                    |3          |
    |2018|3    |Trek Remedy 9.8 27.5 - 2018                          |3          |
    |2018|3    |Electra Townie Original 21D EQ Ladies' - 2018        |3          |
    |2018|3    |Surly Krampus - 2018                                 |3          |
    |2018|3    |Sun Bicycles Lil Kitt'n - 2017                       |3          |
    |2018|3    |Surly ECR 27.5 - 2018                                |3          |
    |2018|3    |Strider Classic 12 Balance Bike - 2018               |3          |
    |2018|3    |Surly Straggler - 2018                               |3          |
    |2018|3    |Surly Straggler 650b - 2018                          |2          |
    |2018|3    |Trek Dual Sport+ - 2018                              |2          |
    |2018|3    |Electra Cruiser Lux 1 Ladies' - 2018                 |2          |
    |2018|3    |Electra Sweet Ride 3i (20-inch) - Girls' - 2018      |2          |
    |2018|3    |Surly Big Fat Dummy Frameset - 2018                  |2          |
    |2018|3    |Trek Silque SLR 7 Women's - 2017                     |2          |
    |2018|3    |Trek Domane S 5 Disc - 2017                          |2          |
    |2018|3    |"Electra Treasure 1 20"" - 2018"                     |2          |
    |2018|3    |Haro Shredder 20 - 2017                              |2          |
    |2018|3    |Trek Silque SLR 8 Women's - 2017                     |2          |
    |2018|3    |Trek Girl's Kickster - 2017                          |2          |
    |2018|3    |Trek Crockett 7 Disc - 2018                          |2          |
    |2018|3    |Trek Precaliber 16 Boys - 2017                       |2          |
    |2018|3    |Strider Strider 20 Sport - 2018                      |2          |
    |2018|3    |"Electra Starship 1 16"" - 2018"                     |2          |
    |2018|3    |"Electra Under-The-Sea 1 16"" - 2018"                |2          |
    |2018|3    |Electra Water Lily 1 (16-inch) - Girl's - 2018       |2          |
    |2018|3    |Trek Precaliber 24 21-speed Girl's - 2018            |2          |
    |2018|3    |Electra Townie Original 7D EQ - Women's - 2016       |2          |
    |2018|3    |Surly Pack Rat - 2018                                |2          |
    |2018|3    |Electra Cruiser Lux 1 - 2016/2018                    |2          |
    |2018|3    |Haro Flightline Two 26 Plus - 2017                   |2          |
    |2018|3    |Sun Bicycles Biscayne Tandem CB - 2017               |2          |
    |2018|3    |Electra Amsterdam Fashion 7i Ladies' - 2017          |2          |
    |2018|3    |Trek Domane ALR 5 Disc - 2018                        |2          |
    |2018|3    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |2          |
    |2018|3    |Trek Powerfly 8 FS Plus - 2017                       |2          |
    |2018|3    |Trek Domane SL 5 Disc Women's - 2018                 |2          |
    |2018|3    |Trek Domane AL 2 - 2018                              |2          |
    |2018|3    |Trek Precaliber 16 Girl's - 2018                     |2          |
    |2018|3    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |2          |
    |2018|3    |Trek Domane SL Disc Frameset - 2017                  |2          |
    |2018|3    |Trek 1120 - 2018                                     |2          |
    |2018|3    |Trek Marlin 7 - 2017/2018                            |2          |
    |2018|3    |Trek Domane ALR 4 Disc Women's - 2018                |2          |
    |2018|3    |Sun Bicycles Drifter 7 - 2017                        |2          |
    |2018|3    |Trek Fuel EX 8 29 XT - 2018                          |2          |
    |2018|3    |Trek Lift+ - 2018                                    |2          |
    |2018|3    |Electra Relic 3i - 2018                              |2          |
    |2018|3    |Electra Townie Balloon 7i EQ Ladies' - 2017/2018     |2          |
    |2018|3    |Trek Fuel EX 7 29 - 2018                             |2          |
    |2018|3    |Electra Townie Commute 27D Ladies - 2018             |2          |
    |2018|3    |Trek Domane ALR Frameset - 2018                      |2          |
    |2018|3    |Electra Townie Original 21D Ladies' - 2018           |2          |
    |2018|3    |Trek Stache 5 - 2017                                 |2          |
    |2018|3    |Electra Glam Punk 3i Ladies' - 2017                  |2          |
    |2018|3    |Electra Townie Original 7D EQ Ladies' - 2017/2018    |2          |
    |2018|3    |Trek Precaliber 24 (21-Speed) - Girls - 2017         |2          |
    |2018|3    |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |2          |
    |2018|3    |Electra Super Moto 8i - 2018                         |2          |
    |2018|3    |Trek Neko+ - 2018                                    |2          |
    |2018|3    |Surly Wednesday - 2017                               |2          |
    |2018|3    |Sun Bicycles Revolutions 24 - Girl's - 2017          |2          |
    |2018|3    |Electra Townie Balloon 3i EQ - 2017/2018             |2          |
    |2018|3    |Electra Cruiser Lux Fat Tire 7D - 2018               |2          |
    |2018|3    |Trek Emonda SLR 6 - 2018                             |2          |
    |2018|3    |Sun Bicycles Spider 3i - 2017                        |2          |
    |2018|3    |Sun Bicycles Brickell Tandem CB - 2017               |2          |
    |2018|3    |Trek Slash 8 27.5 - 2016                             |2          |
    |2018|3    |Trek Emonda SL 6 Disc - 2018                         |2          |
    |2018|3    |Haro Shredder 20 Girls - 2017                        |1          |
    |2018|3    |Trek X-Caliber 8 - 2017                              |1          |
    |2018|3    |Trek CrossRip+ - 2018                                |1          |
    |2018|3    |Electra Cruiser 1 Ladies' - 2018                     |1          |
    |2018|3    |Sun Bicycles Drifter 7 - Women's - 2017              |1          |
    |2018|3    |Trek Emonda S 5 - 2017                               |1          |
    |2018|3    |Electra Townie Original 7D - 2015/2016               |1          |
    |2018|3    |Electra Townie Original 3i EQ Ladies' - 2018         |1          |
    |2018|3    |Trek Precaliber 16 Girls - 2017                      |1          |
    |2018|3    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |1          |
    |2018|3    |Trek Precaliber 20 6-speed Boy's - 2018              |1          |
    |2018|3    |Surly Ice Cream Truck Frameset - 2016                |1          |
    |2018|3    |Trek Emonda S 4 - 2017                               |1          |
    |2018|3    |Trek Emonda ALR 6 - 2018                             |1          |
    |2018|3    |Trek Remedy 7 27.5 - 2018                            |1          |
    |2018|3    |Electra Cruiser 7D Ladies' - 2016/2018               |1          |
    |2018|3    |Trek Boone 7 Disc - 2018                             |1          |
    |2018|3    |"Electra Girl's Hawaii 1 16"" - 2017"                |1          |
    |2018|3    |Sun Bicycles Streamway 7 - 2017                      |1          |
    |2018|3    |Pure Cycles Vine 8-Speed - 2016                      |1          |
    |2018|3    |Electra Townie Original 3i EQ - 2017/2018            |1          |
    |2018|3    |Trek Domane SL 5 Disc - 2018                         |1          |
    |2018|3    |Electra Straight 8 1 (16-inch) - Boy's - 2018        |1          |
    |2018|3    |Trek Verve+ Lowstep - 2018                           |1          |
    |2018|3    |Trek Madone 9.2 - 2017                               |1          |
    |2018|3    |Electra Heartchya 1 (20-inch) - Girl's - 2018        |1          |
    |2018|3    |Electra White Water 3i - 2018                        |1          |
    |2018|3    |Trek Precaliber 12 Boy's - 2018                      |1          |
    |2018|3    |Surly ECR Frameset - 2018                            |1          |
    |2018|3    |Electra Queen of Hearts 3i - 2018                    |1          |
    |2018|3    |Trek Fuel EX 5 27.5 Plus - 2017                      |1          |
    |2018|3    |Haro SR 1.3 - 2017                                   |1          |
    |2018|3    |Electra Tiger Shark 3i (20-inch) - Boys' - 2018      |1          |
    |2018|3    |Surly Steamroller - 2017                             |1          |
    |2018|3    |Trek Conduit+ - 2016                                 |1          |
    |2018|3    |Trek Ticket S Frame - 2018                           |1          |
    |2018|3    |Heller Shagamaw GX1 - 2018                           |1          |
    |2018|3    |Electra Loft Go! 8i - 2018                           |1          |
    |2018|3    |Electra Townie Original 21D - 2018                   |1          |
    |2018|3    |Trek Domane SL 8 Disc - 2018                         |1          |
    |2018|3    |Electra Townie Original 1 - 2018                     |1          |
    |2018|3    |Sun Bicycles Atlas X-Type - 2017                     |1          |
    |2018|3    |Sun Bicycles ElectroLite - 2017                      |1          |
    |2018|3    |Electra Cruiser Lux 3i - 2018                        |1          |
    |2018|3    |Trek Domane SLR 8 Disc - 2018                        |1          |
    |2018|3    |Surly Karate Monkey 27.5+ Frameset - 2017            |1          |
    |2018|3    |Trek Procaliber 6 - 2018                             |1          |
    |2018|3    |Electra Delivery 3i - 2016/2017/2018                 |1          |
    |2018|3    |Surly Ogre Frameset - 2017                           |1          |
    |2018|3    |Electra Townie Original 21D - 2016                   |1          |
    |2018|3    |Trek Domane S 6 - 2017                               |1          |
    |2018|3    |Electra Straight 8 3i (20-inch) - Boy's - 2017       |1          |
    |2018|3    |Trek Powerfly 5 FS - 2018                            |1          |
    |2018|3    |Trek Fuel EX 8 29 - 2018                             |1          |
    |2018|3    |Electra Amsterdam Royal 8i - 2017/2018               |1          |
    |2018|4    |Electra Townie Original 21D EQ - 2017/2018           |12         |
    |2018|4    |Electra Townie Commute Go! - 2018                    |8          |
    |2018|4    |Electra Townie Commute 27D Ladies - 2018             |7          |
    |2018|4    |Electra Townie Original 21D - 2016                   |7          |
    |2018|4    |Trek Domane SL 6 - 2017                              |7          |
    |2018|4    |Haro Flightline One ST - 2017                        |7          |
    |2018|4    |Electra Townie Balloon 3i EQ Ladies' - 2018          |7          |
    |2018|4    |Electra Super Moto 8i - 2018                         |6          |
    |2018|4    |Electra Townie Commute Go! Ladies' - 2018            |6          |
    |2018|4    |Strider Strider 20 Sport - 2018                      |6          |
    |2018|4    |"Electra Treasure 1 20"" - 2018"                     |6          |
    |2018|4    |Haro Shift R3 - 2017                                 |6          |
    |2018|4    |Trek Girl's Kickster - 2017                          |6          |
    |2018|4    |Haro Shredder 20 - 2017                              |6          |
    |2018|4    |Trek Boone 7 Disc - 2018                             |6          |
    |2018|4    |Electra Savannah 3i (20-inch) - Girl's - 2017        |6          |
    |2018|4    |Trek Conduit+ - 2018                                 |6          |
    |2018|4    |"Electra Girl's Hawaii 1 16"" - 2017"                |5          |
    |2018|4    |Surly Troll Frameset - 2018                          |5          |
    |2018|4    |Surly Big Fat Dummy Frameset - 2018                  |5          |
    |2018|4    |Trek CrossRip 1 - 2018                               |5          |
    |2018|4    |Trek Kids' Neko - 2018                               |5          |
    |2018|4    |Electra Townie Original 7D EQ Ladies' - 2017/2018    |5          |
    |2018|4    |Trek Super Commuter+ 7 - 2018                        |5          |
    |2018|4    |Trek Domane ALR Disc Frameset - 2018                 |5          |
    |2018|4    |Trek Verve+ - 2018                                   |5          |
    |2018|4    |Trek Powerfly 5 Women's - 2018                       |5          |
    |2018|4    |Sun Bicycles Cruz 3 - 2017                           |5          |
    |2018|4    |Electra Moto 1 - 2016                                |4          |
    |2018|4    |Electra Townie Commute 8D - 2018                     |4          |
    |2018|4    |Surly Pack Rat - 2018                                |4          |
    |2018|4    |Electra Cyclosaurus 1 (16-inch) - Boy's - 2018       |4          |
    |2018|4    |Electra Koa 3i Ladies' - 2018                        |4          |
    |2018|4    |Trek Powerfly 8 FS Plus - 2017                       |4          |
    |2018|4    |Electra Townie Commute 8D Ladies' - 2018             |4          |
    |2018|4    |Surly Krampus - 2018                                 |4          |
    |2018|4    |Electra Delivery 3i - 2016/2017/2018                 |4          |
    |2018|4    |Sun Bicycles Cruz 7 - 2017                           |4          |
    |2018|4    |Electra Cruiser 1 Ladies' - 2018                     |4          |
    |2018|4    |Trek Fuel EX 9.8 29 - 2017                           |4          |
    |2018|4    |Haro SR 1.3 - 2017                                   |4          |
    |2018|4    |Trek Precaliber 16 Girl's - 2018                     |4          |
    |2018|4    |Electra Townie Original 1 - 2018                     |4          |
    |2018|4    |Strider Classic 12 Balance Bike - 2018               |4          |
    |2018|4    |Electra Townie Balloon 8D EQ Ladies' - 2016/2017/2018|4          |
    |2018|4    |Sun Bicycles Atlas X-Type - 2017                     |4          |
    |2018|4    |Electra Tiger Shark 1 (20-inch) - Boys' - 2018       |4          |
    |2018|4    |Sun Bicycles ElectroLite - 2017                      |4          |
    |2018|4    |Trek Fuel EX 8 29 XT - 2018                          |4          |
    |2018|4    |Trek Precaliber 20 Girl's - 2018                     |4          |
    |2018|4    |Sun Bicycles Biscayne Tandem CB - 2017               |4          |
    |2018|4    |Electra Girl's Hawaii 1 (16-inch) - 2015/2016        |3          |
    |2018|4    |Trek Stache 5 - 2018                                 |3          |
    |2018|4    |Sun Bicycles Streamway - 2017                        |3          |
    |2018|4    |Sun Bicycles Boardwalk (24-inch Wheels) - 2017       |3          |
    |2018|4    |Electra Cruiser 1 - 2016/2017/2018                   |3          |
    |2018|4    |Trek Emonda ALR 6 - 2018                             |3          |
    |2018|4    |Trek Domane ALR 4 Disc Women's - 2018                |3          |
    |2018|4    |"Electra Under-The-Sea 1 16"" - 2018"                |3          |
    |2018|4    |Electra Queen of Hearts 3i - 2018                    |3          |
    |2018|4    |Trek XM700+ - 2018                                   |3          |
    |2018|4    |Haro Shredder 20 Girls - 2017                        |3          |
    |2018|4    |Trek Fuel EX 9.8 27.5 Plus - 2017                    |3          |
    |2018|4    |Electra Amsterdam Original 3i Ladies' - 2017         |3          |
    |2018|4    |Electra Cruiser Lux 1 Ladies' - 2018                 |3          |
    |2018|4    |Electra Moto 3i (20-inch) - Boy's - 2017             |3          |
    |2018|4    |Surly ECR 27.5 - 2018                                |3          |
    |2018|4    |Electra Townie Balloon 7i EQ Ladies' - 2017/2018     |3          |
    |2018|4    |Trek Fuel EX 8 29 - 2018                             |3          |
    |2018|4    |Trek Precaliber 12 Boy's - 2018                      |3          |
    |2018|4    |Electra Amsterdam Royal 8i Ladies - 2018             |3          |
    |2018|4    |Electra Townie Original 21D EQ Ladies' - 2018        |3          |
    |2018|4    |Trek Domane SL Frameset - 2018                       |3          |
    |2018|4    |Trek Domane SL Disc Frameset - 2017                  |3          |
    |2018|4    |Trek Kickster - 2018                                 |3          |
    |2018|4    |Trek Domane SLR 9 Disc - 2018                        |3          |
    |2018|4    |Sun Bicycles Lil Bolt Type-R - 2017                  |3          |
    |2018|4    |Trek XM700+ Lowstep - 2018                           |3          |
    |2018|4    |Surly Steamroller - 2017                             |3          |
    |2018|4    |Pure Cycles Vine 8-Speed - 2016                      |3          |
    |2018|4    |Trek Ticket S Frame - 2018                           |3          |
    |2018|4    |Electra Cruiser Lux Fat Tire 1 Ladies - 2017         |3          |
    |2018|4    |Haro SR 1.2 - 2017                                   |3          |
    |2018|4    |Trek Domane SL 6 Disc - 2018                         |3          |
    |2018|4    |"Electra Starship 1 16"" - 2018"                     |3          |
    |2018|4    |Electra Cruiser Lux 7D - 2018                        |3          |
    |2018|4    |Trek Domane ALR 5 Gravel - 2018                      |2          |
    |2018|4    |Electra Townie Commute 27D - 2018                    |2          |
    |2018|4    |Electra Townie Original 7D EQ - 2016                 |2          |
    |2018|4    |Trek Fuel EX 5 Plus - 2018                           |2          |
    |2018|4    |Electra Townie 7D (20-inch) - Boys' - 2017           |2          |
    |2018|4    |Trek Precaliber 12 Girls - 2017                      |2          |
    |2018|4    |Trek Verve+ Lowstep - 2018                           |2          |
    |2018|4    |Electra Townie Balloon 7i EQ - 2018                  |2          |
    |2018|4    |Trek Stache 5 - 2017                                 |2          |
    |2018|4    |Trek Marlin 5 - 2018                                 |2          |
    |2018|4    |Trek Precaliber 24 (7-Speed) - Boys - 2018           |2          |
    |2018|4    |Electra Townie Go! 8i - 2017/2018                    |2          |
    |2018|4    |Strider Sport 16 - 2018                              |2          |
    |2018|4    |Trek Domane SL 5 Disc - 2018                         |2          |
    |2018|4    |Haro Downtown 16 - 2017                              |2          |
    |2018|4    |Electra Cruiser Lux 3i Ladies' - 2018                |2          |
    |2018|4    |Sun Bicycles Drifter 7 - Women's - 2017              |2          |
    |2018|4    |Electra Townie Original 7D - 2017                    |2          |
    |2018|4    |Surly Big Dummy Frameset - 2017                      |2          |
    |2018|4    |Electra Amsterdam Fashion 3i Ladies' - 2017/2018     |2          |
    |2018|4    |Trek Super Commuter+ 8S - 2018                       |2          |
    |2018|4    |Electra Straight 8 3i - 2018                         |2          |
    |2018|4    |Trek Domane ALR 4 Disc - 2018                        |2          |
    |2018|4    |Heller Bloodhound Trail - 2018                       |2          |
    |2018|4    |Electra Relic 3i - 2018                              |2          |
    |2018|4    |Trek Domane SLR 6 Disc - 2018                        |2          |
    |2018|4    |Trek Precaliber 20 Boy's - 2018                      |2          |
    |2018|4    |Electra Townie Original 7D EQ - Women's - 2016       |2          |
    |2018|4    |Trek Crockett 7 Disc - 2018                          |2          |
    |2018|4    |Trek Domane SLR 6 - 2018                             |2          |
    |2018|4    |Trek Domane AL 2 - 2018                              |2          |
    |2018|4    |Trek Madone 9.2 - 2017                               |2          |
    |2018|4    |Trek Domane SL 5 - 2018                              |2          |
    |2018|4    |Electra Townie Original 1 Ladies' - 2018             |2          |
    |2018|4    |Trek Boone 7 - 2017                                  |2          |
    |2018|4    |Trek Domane SL 6 - 2018                              |2          |
    |2018|4    |Trek Remedy 7 27.5 - 2018                            |2          |
    |2018|4    |Electra Tiger Shark 3i - 2018                        |2          |
    |2018|4    |Sun Bicycles Cruz 3 - Women's - 2017                 |2          |
    |2018|4    |Pure Cycles William 3-Speed - 2016                   |2          |
    |2018|4    |Trek X-Caliber Frameset - 2018                       |2          |
    |2018|4    |Trek X-Caliber 8 - 2018                              |2          |
    |2018|4    |Trek Domane SLR Frameset - 2018                      |2          |
    |2018|4    |Electra Water Lily 1 (16-inch) - Girl's - 2018       |2          |
    |2018|4    |Electra Cruiser Lux 3i - 2018                        |2          |
    |2018|4    |Sun Bicycles Streamway 7 - 2017                      |2          |
    |2018|4    |Electra Townie Go! 8i Ladies' - 2018                 |2          |
    |2018|4    |Trek Farley Carbon Frameset - 2018                   |2          |
    |2018|4    |Trek Emonda SLR 8 - 2018                             |2          |
    |2018|4    |Surly Straggler 650b - 2016                          |2          |
    |2018|4    |Trek Domane SL 7 Women's - 2018                      |2          |
    |2018|4    |Electra Townie Original 21D Ladies' - 2018           |2          |
    |2018|4    |Surly Straggler - 2018                               |2          |
    |2018|4    |Electra Sweet Ride 3i (20-inch) - Girls' - 2018      |2          |
    |2018|4    |Trek Marlin 6 - 2018                                 |2          |
    |2018|4    |Trek Silque SLR 7 Women's - 2017                     |2          |
    |2018|4    |Trek Domane AL 2 Women's - 2018                      |2          |
    |2018|4    |Trek Emonda S 5 - 2017                               |2          |
    |2018|4    |Trek Conduit+ - 2016                                 |2          |
    |2018|4    |Surly Straggler 650b - 2018                          |2          |
    |2018|4    |Electra Cruiser Lux 1 - 2016/2018                    |2          |
    |2018|4    |Electra Girl's Hawaii 1 (20-inch) - 2015/2016        |2          |
    |2018|4    |Trek Domane SLR 8 Disc - 2018                        |2          |
    |2018|4    |Pure Cycles Western 3-Speed - Women's - 2015/2016    |2          |
    |2018|4    |Electra Townie Original 3i EQ Ladies' - 2018         |2          |
    |2018|4    |Trek Powerfly 7 FS - 2018                            |2          |
    |2018|4    |Electra White Water 3i - 2018                        |2          |
    |2018|4    |Trek Slash 8 27.5 - 2016                             |2          |
    |2018|4    |Trek Procaliber Frameset - 2018                      |2          |
    |2018|4    |Heller Shagamaw GX1 - 2018                           |2          |
    |2018|4    |Surly Troll Frameset - 2017                          |2          |
    |2018|4    |Trek Boone Race Shop Limited - 2017                  |2          |
    |2018|4    |Surly Ice Cream Truck Frameset - 2016                |2          |
    |2018|4    |Surly Ogre Frameset - 2017                           |2          |
    |2018|4    |Surly Karate Monkey 27.5+ Frameset - 2017            |2          |
    |2018|4    |Trek Dual Sport+ - 2018                              |2          |
    |2018|4    |Trek Precaliber 20 6-speed Boy's - 2018              |1          |
    |2018|4    |Electra Townie Original 3i EQ - 2017/2018            |1          |
    |2018|4    |Sun Bicycles Revolutions 24 - Girl's - 2017          |1          |
    |2018|4    |Sun Bicycles Revolutions 24 - 2017                   |1          |
    |2018|4    |Trek Crockett 5 Disc - 2018                          |1          |
    |2018|4    |Trek Domane SL 5 Women's - 2018                      |1          |
    |2018|4    |Trek Emonda SL 7 - 2018                              |1          |
    |2018|4    |Electra Morningstar 3i Ladies' - 2018                |1          |
    |2018|4    |Electra Townie Original 21D - 2018                   |1          |
    |2018|4    |Trek Farley Alloy Frameset - 2017                    |1          |
    |2018|4    |Trek Domane ALR Frameset - 2018                      |1          |
    |2018|4    |Trek Stache Carbon Frameset - 2018                   |1          |
    |2018|4    |Trek Superfly 24 - 2017/2018                         |1          |
    |2018|4    |Electra Soft Serve 1 (16-inch) - Girl's - 2018       |1          |
    |2018|4    |Electra Townie Balloon 3i EQ - 2017/2018             |1          |
    |2018|4    |Trek Emonda S 4 - 2017                               |1          |
    |2018|4    |Trek Procaliber 6 - 2018                             |1          |
    |2018|4    |Trek 820 - 2018                                      |1          |
    |2018|4    |Trek Madone 9 Frameset - 2018                        |1          |
    |2018|4    |Sun Bicycles Cruz 7 - Women's - 2017                 |1          |
    |2018|4    |Trek Neko+ - 2018                                    |1          |
    |2018|4    |Trek Domane AL 3 - 2018                              |1          |
    |2018|4    |Electra Straight 8 1 (20-inch) - Boy's - 2018        |1          |
    |2018|4    |Trek Session DH 27.5 Carbon Frameset - 2017          |1          |
    |2018|4    |Surly Straggler - 2016                               |1          |
    |2018|4    |Trek Precaliber 24 7-speed Girl's - 2018             |1          |
    |2018|4    |Electra Cruiser 7D (24-Inch) Ladies' - 2016/2018     |1          |
    |2018|4    |Electra Cruiser 7D Ladies' - 2016/2018               |1          |
    |2018|4    |Haro SR 1.1 - 2017                                   |1          |
    |2018|4    |Electra Moto 3i - 2018                               |1          |
    |2018|4    |Electra Cruiser 7D - 2016/2017/2018                  |1          |
    |2018|4    |Trek Domane SL 8 Disc - 2018                         |1          |
    |2018|4    |Electra Straight 8 3i (20-inch) - Boy's - 2017       |1          |
    |2018|4    |Trek Domane S 6 - 2017                               |1          |
    |2018|4    |Trek CrossRip+ - 2018                                |1          |
    |2018|4    |Electra Townie 3i EQ (20-inch) - Boys' - 2017        |1          |
    |2018|4    |Trek Boone 5 Disc - 2018                             |1          |
    |2018|4    |Electra Loft Go! 8i - 2018                           |1          |
    |2018|4    |Trek Lift+ - 2018                                    |1          |
    |2018|4    |Electra Townie Original 7D - 2015/2016               |1          |
    |2018|4    |Trek X-Caliber 7 - 2018                              |1          |
    |2018|4    |Trek Domane SLR 6 Disc - 2017                        |1          |
    |2018|4    |Ritchey Timberwolf Frameset - 2016                   |1          |
    |2018|4    |Surly Pack Rat Frameset - 2018                       |1          |
    |2018|4    |Trek Marlin 7 - 2017/2018                            |1          |
    |2018|4    |Trek Silque SLR 8 Women's - 2017                     |1          |
    |2018|4    |Trek Domane S 5 Disc - 2017                          |1          |
    |2018|4    |Trek Domane SLR Disc Frameset - 2018                 |1          |
    |2018|4    |Trek Emonda SLR 6 - 2018                             |1          |
    |2018|4    |Trek Precaliber 24 21-speed Boy's - 2018             |1          |
    |2018|4    |Electra Townie Original 7D EQ - 2018                 |1          |
    |2018|4    |Electra Townie Balloon 8D EQ - 2016/2017/2018        |1          |
    |2018|4    |Sun Bicycles Drifter 7 - 2017                        |1          |
    |2018|4    |Trek Domane AL 3 Women's - 2018                      |1          |
    |2018|4    |Trek Fuel EX 8 29 - 2016                             |1          |
    |2018|4    |Trek Precaliber 24 (21-Speed) - Girls - 2017         |1          |
    |2018|4    |Trek Fuel EX 5 27.5 Plus - 2017                      |1          |
    |2018|4    |Surly Wednesday Frameset - 2017                      |1          |
    |2018|4    |Sun Bicycles Streamway 3 - 2017                      |1          |
    |2018|4    |Trek Precaliber 16 Boys - 2017                       |1          |
    |2018|4    |Electra Sugar Skulls 1 (20-inch) - Girl's - 2017     |1          |
    |2018|6    |Trek Precaliber 16 Girl's - 2018                     |1          |
    |2018|7    |Trek Procal AL Frameset - 2018                       |2          |
    |2018|7    |Trek X-Caliber 8 - 2017                              |2          |
    |2018|7    |Electra Cruiser Lux 3i Ladies' - 2018                |2          |
    |2018|7    |Electra Townie Balloon 7i EQ - 2018                  |1          |
    |2018|7    |Trek CrossRip+ - 2018                                |1          |
    |2018|7    |Electra Townie Original 3i EQ - 2017/2018            |1          |
    |2018|7    |Sun Bicycles Biscayne Tandem 7 - 2017                |1          |
    |2018|7    |Trek Precaliber 16 Girl's - 2018                     |1          |
    |2018|8    |Trek Domane ALR Frameset - 2018                      |2          |
    |2018|8    |Electra Cruiser 7D Tall - 2016/2018                  |2          |
    |2018|8    |Electra Townie Balloon 8D EQ - 2016/2017/2018        |2          |
    |2018|8    |Electra Moto 3i (20-inch) - Boy's - 2017             |1          |
    |2018|8    |Sun Bicycles Streamway 7 - 2017                      |1          |
    |2018|8    |Surly Troll Frameset - 2017                          |1          |
    |2018|9    |Trek Domane SL 6 - 2018                              |2          |
    |2018|9    |Electra Loft Go! 8i - 2018                           |1          |
    |2018|9    |Electra Morningstar 3i Ladies' - 2018                |1          |
    |2018|10   |"Electra Superbolt 1 20"" - 2018"                    |2          |
    |2018|10   |Electra Townie 7D (20-inch) - Boys' - 2017           |2          |
    |2018|10   |Electra Tiger Shark 1 (20-inch) - Boys' - 2018       |2          |
    |2018|10   |Electra Townie Commute 8D Ladies' - 2018             |1          |
    |2018|10   |Sun Bicycles ElectroLite - 2017                      |1          |
    |2018|11   |Electra Cruiser 1 - 2016/2017/2018                   |2          |
    |2018|11   |Trek Emonda ALR 6 - 2018                             |2          |
    |2018|11   |Electra Heartchya 1 (20-inch) - Girl's - 2018        |2          |
    |2018|11   |Trek Domane SL 7 Women's - 2018                      |1          |
    |2018|11   |Surly Krampus - 2018                                 |1          |
    |2018|12   |Trek Verve+ Lowstep - 2018                           |2          |
    |2018|12   |Trek Domane SL 5 Disc - 2018                         |1          |
    |2018|12   |Electra Tiger Shark 3i - 2018                        |1          |
    +----+-----+-----------------------------------------------------+-----------+
    
    


```python
import pandas as pd
import matplotlib.pyplot as plt
```


```python
# İlgili ürün adını girin
product_name = "Electra Cruiser 1 (24-Inch) - 2016"

```


```python
# İlgili ürünün satış verilerini alın
product_data = merged_data.filter(merged_data.product_name == product_name).orderBy("year", "month")
```


```python
# Satış verilerini Pandas DataFrame'e dönüştürün
product_df = product_data.groupBy("year", "month").agg(F.sum("quantity").alias("total_sales")).orderBy("year", "month").toPandas()
```


```python
# Toplam satış miktarını hesaplayın
total_sales = product_df["total_sales"].sum()
```


```python
# Yıl ve ay sütunlarını bir tarih sütunu olarak birleştirin
product_df["date"] = pd.to_datetime(product_df["year"].astype(str) + "-" + product_df["month"].astype(str), format="%Y-%m")

```


```python
# Tarih sütununu indeks olarak ayarlayın
product_df.set_index("date", inplace=True)
```


```python
# Satış verilerini çizmek için bir çubuk grafiği oluşturun
plt.figure(figsize=(16, 6))
plt.bar(product_df.index, product_df["total_sales"], width=20, color='b', alpha=0.7)
plt.xlabel("Tarih")
plt.ylabel("Satış Miktarı")
plt.title(f"{product_name} Satışları (Toplam: {total_sales} adet)")
plt.grid(axis="y", linestyle="--", alpha=0.7)
plt.xticks(rotation=45)
plt.tight_layout()
```


    
![png](output_16_0.png)
    



```python

```
