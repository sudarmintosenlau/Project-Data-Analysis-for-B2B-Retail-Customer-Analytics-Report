# Project-Data-Analysis-for-B2B-Retail-Customer-Analytics-Report
Project dari DQLab Melakukan analisis Performance dengan menggunakan MySQL

xyz.com adalah perusahan rintisan B2B yang menjual berbagai produk tidak langsung kepada end user tetapi ke bisnis/perusahaan lainnya. Sebagai data-driven company, maka setiap pengambilan keputusan di xyz.com selalu berdasarkan data. Setiap quarter xyz.com akan mengadakan townhall dimana seluruh atau perwakilan divisi akan berkumpul untuk me-review performance perusahaan selama quarter terakhir.

Beberapa pertanyaan yang dijawab pada customer analytics report ini
* Bagaimana pertumbuhan penjualan saat ini?
* Apakah jumlah customers xyz.com semakin bertambah ?
* Dan seberapa banyak customers tersebut yang sudah melakukan transaksi?
* Category produk apa saja yang paling banyak dibeli oleh customers?
* Seberapa banyak customers yang tetap aktif bertransaksi?

Tabel yang akan digunakan pada project kali ini adalah sebagai berikut (Asumsikan tahun yang sedang berjalan adalah tahun 2004).

![image](https://user-images.githubusercontent.com/62486840/147630775-f2d6b5ef-2382-4fa0-a794-a1764d417c3b.png)

Tabel orders_1 : Berisi data terkait transaksi penjualan periode quarter 1 (Jan – Mar 2004)

Tabel Orders_2 : Berisi data terkait transaksi penjualan periode quarter 2 (Apr – Jun 2004)

Tabel Customer : Berisi data profil customer yang mendaftar menjadi customer xyz.com

**Memahami table**

Sebelum memulai menyusun query SQL dan membuat Analisa dari hasil query, hal pertama yang perlu dilakukan adalah menjadi familiar dengan tabel yang akan digunakan. Hal ini akan sangat berguna dalam menentukan kolom mana sekiranya berkaitan dengan problem yang akan dianalisa, dan proses manipulasi data apa yang sekiranya perlu dilakukan untuk kolom – kolom tersebut, karena tidak semua kolom pada tabel perlu untuk digunakan.
```
SELECT * FROM orders_1 limit 5;
SELECT * FROM orders_2 limit 5;
SELECT * FROM customer limit 5;
```
keluaran:

![image](https://user-images.githubusercontent.com/62486840/147630384-f330b5b5-9695-4cc3-86bd-d6d36e63c400.png)


### 1. Total Penjualan dan Revenue pada Quarter-1 (Jan, Feb, Mar) dan Quarter-2 (Apr,Mei,Jun)
* Dari tabel orders_1 lakukan penjumlahan pada kolom quantity dengan fungsi aggregate sum() dan beri nama “total_penjualan”, kalikan kolom quantity dengan kolom priceEach kemudian jumlahkan hasil perkalian kedua kolom tersebut dan beri nama “revenue”.
* Perusahaan hanya ingin menghitung penjualan dari produk yang terkirim saja, jadi kita perlu mem-filter kolom ‘status’ sehingga hanya menampilkan order dengan status “Shipped”.
* Lakukan Langkah 1 & 2, untuk tabel orders_2.
```
SELECT 
	sum(quantity) as total_penjualan,
	sum(quantity*priceeach) as revenue 
FROM 
	orders_1;
SELECT 
	sum(quantity) as total_penjualan,
	sum(quantity*priceeach) as revenue 
FROM 
	orders_2 
WHERE 
	status = 'Shipped';
```
keluaran:

![image](https://user-images.githubusercontent.com/62486840/147631292-602c1c1f-4f4c-46b0-8596-e34bb1381fe6.png)

### 2. Menghitung persentasi keseluruhan penjualan
Dikarenakan kedua tabel orders_1 dan orders_2 masih terpisah, untuk menghitung persentasi keseluruhan penjualan dari kedua tabel tersebut perlu digabungkan :

* Pilihlah kolom “orderNumber”, “status”, “quantity”, “priceEach” pada tabel orders_1, dan tambahkan kolom baru dengan nama “quarter” dan isi dengan value “1”. Lakukan yang sama dengan tabel orders_2, dan isi dengan value “2”, kemudian gabungkan kedua tabel tersebut.
* Gunakan statement dari Langkah 1 sebagai subquery dan beri alias “tabel_a”.
* Dari “tabel_a”, lakukan penjumlahan pada kolom “quantity” dengan fungsi aggregate sum() dan beri nama “total_penjualan”, dan kalikan kolom quantity dengan kolom priceEach kemudian jumlahkan hasil perkalian kedua kolom tersebut dan beri nama “revenue”
* Filter kolom ‘status’ sehingga hanya menampilkan order dengan status “Shipped”.
* Kelompokkan total_penjualan berdasarkan kolom “quarter”, dan jangan lupa menambahkan kolom ini pada bagian select.
```
SELECT 
	quarter,
	sum(quantity) as total_penjualan,
	sum(quantity*priceeach) as revenue 
FROM 
	(select ordernumber,status,quantity,priceeach, 1 as quarter from orders_1
UNION
	select ordernumber,status,quantity,priceeach, 2 as quarter from orders_2) as tabel_a 
WHERE 
	status = 'Shipped'
GROUP BY 
	quarter;
```
keluaran:

![image](https://user-images.githubusercontent.com/62486840/147631784-bc78683e-2ca2-490c-8ccb-b4e0160e7008.png)

sehingga perhitungan pertumbuhan penjualan adalah sebagai berikut:

**%Growth Penjualan** = (6717 – 8694)/8694 = -22%
**%Growth Revenue**   = (607548320 – 799579310)/ 799579310 = -24%

### 3. Apakah jumlah customers xyz.com semakin bertambah?
Penambahan jumlah customers dapat diukur dengan membandingkan total jumlah customers yang registrasi di periode saat ini dengan total jumlah customers yang registrasi diakhir periode sebelumnya.
```
SELECT 
	quarter(createdate) as quarter, 
	count(distinct customerid) as total_customers 
FROM 
	(SELECT 
     		customerid,
		createdate,
		quarter(createdate) as quarter 
     	FROM customer 
WHERE 
	createdate between '2004-01-01' and '2004-06-30') as tabel_b
GROUP BY 
	quarter(createdate);
```
keluaran:

![image](https://user-images.githubusercontent.com/62486840/147632602-8a4f983d-93b5-4468-ac77-069f1bb6787e.png)

### 4. Seberapa banyak customers tersebut yang sudah melakukan transaksi?
Problem ini merupakan kelanjutan dari problem sebelumnya yaitu dari sejumlah customer yang registrasi di periode quarter-1 dan quarter-2, berapa banyak yang sudah melakukan transaksi
```
SELECT 
	quarter(createdate) as quarter, 
    count(distinct customerid) as total_customers 
FROM 
	(SELECT 
     	customerid,
     	createdate,
     	quarter(createdate) as quarter 
     FROM customer 
     WHERE createdate between '2004-01-01' and '2004-06-30' and customerid in 
 (select distinct customerid from orders_1
UNION
select distinct customerid from orders_2)) as tabel_b
GROUP BY 
	quarter(createdate);
```
keluaran:

![image](https://user-images.githubusercontent.com/62486840/147632932-937d37c1-131d-49aa-9963-b87e64dd2c5f.png)

### 5. Category produk apa saja yang paling banyak di-order oleh customers di Quarter-2?
Untuk mengetahui kategori produk yang paling banyak dibeli, maka dapat dilakukan dengan menghitung total order dan jumlah penjualan dari setiap kategori produk.
```
SELECT 
	left(productcode,3) as categoryid,
    count(distinct ordernumber) as total_order,
    sum(quantity) as total_penjualan 
FROM 
	(select productcode,ordernumber,quantity,status,left(productcode,3) as categoryid from orders_2 WHERE 
    status = "Shipped") tabel_c
GROUP BY 
	left(productcode,3)
ORDER BY 
	count(distinct ordernumber)
DESC;
```
keluaran:

![image](https://user-images.githubusercontent.com/62486840/147633297-986b61f5-4f64-4e18-9d43-5a554c3c410c.png)

### 6. Seberapa banyak customers yang tetap aktif bertransaksi setelah transaksi pertamanya?
Mengetahui seberapa banyak customers yang tetap aktif menunjukkan apakah xyz.com tetap digemari oleh customers untuk memesan kebutuhan bisnis mereka. Hal ini juga dapat menjadi dasar bagi tim product dan business untuk pengembangan product dan business kedepannya. Adapun metrik yang digunakan disebut retention cohort. Untuk project ini, kita akan menghitung retention dengan query SQL sederhana.

Oleh karena baru terdapat 2 periode yang Quarter 1 dan Quarter 2, maka retention yang dapat dihitung adalah retention dari customers yang berbelanja di Quarter 1 dan kembali berbelanja di Quarter 2, sedangkan untuk customers yang berbelanja di Quarter 2 baru bisa dihitung retentionnya di Quarter 3.
```
#Menghitung total unik customers yang transaksi di quarter_1
SELECT 
	COUNT(DISTINCT customerID) as total_customers 
FROM orders_1;
#output = 25
SELECT "1" as quarter, (COUNT(DISTINCT customerID)*100)/25 as q2 
FROM orders_1 where customerid in (SELECT DISTINCT customerID FROM orders_2);
```
keluaran:

![image](https://user-images.githubusercontent.com/62486840/147633543-347e58c5-e92b-400a-9bc4-ca767932abe6.png)

### Kesimpulan
Berdasarkan data yang telah kita peroleh melalui query SQL, Kita dapat menarik kesimpulan bahwa :
1. Performance xyz.com menurun signifikan di quarter ke-2, terlihat dari nilai penjualan dan revenue yang drop hingga 20% dan 24%,
2. perolehan customer baru juga tidak terlalu baik, dan sedikit menurun dibandingkan quarter sebelumnya.
3. Ketertarikan customer baru untuk berbelanja di xyz.com masih kurang, hanya sekitar 56% saja yang sudah bertransaksi. Disarankan tim Produk untuk perlu mempelajari behaviour customer dan melakukan product improvement, sehingga conversion rate (register to transaction) dapat meningkat.
4. Produk kategori S18 dan S24 berkontribusi sekitar 50% dari total order dan 60% dari total penjualan, sehingga xyz.com sebaiknya fokus untuk pengembangan category S18 dan S24.
5. Retention rate customer xyz.com juga sangat rendah yaitu hanya 24%, artinya banyak customer yang sudah bertransaksi di quarter-1 tidak kembali melakukan order di quarter ke-2 (no repeat order).
6. xyz.com mengalami pertumbuhan negatif di quarter ke-2 dan perlu melakukan banyak improvement baik itu di sisi produk dan bisnis marketing, jika ingin mencapai target dan positif growth di quarter ke-3. Rendahnya retention rate dan conversion rate bisa menjadi diagnosa awal bahwa customer tidak tertarik/kurang puas/kecewa berbelanja di xyz.com.



Project ini dapat diakses melalui https://academy.dqlab.id/main/package/practice/246.

![image](https://user-images.githubusercontent.com/62486840/147630649-10e8c741-dc06-459a-87df-ee61434cb6e3.png)
