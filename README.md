# Project-Data-Analysis-for-B2B-Retail-Customer-Analytics-Report
Project dari DQLab Melakukan analisis Performance dengan menggunakan MySQL


Tabel yang Digunakan
Tabel yang akan digunakan pada project kali ini adalah sebagai berikut.

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







































Project ini dapat diakses melalui https://academy.dqlab.id/main/package/practice/246.

![image](https://user-images.githubusercontent.com/62486840/147630649-10e8c741-dc06-459a-87df-ee61434cb6e3.png)
