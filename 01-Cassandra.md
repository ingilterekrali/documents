**Author**: Akın Abdullahoğlu

**Tarih**: 24 Temmuz 2017

# Konsol üzerinden Cassandra’ya bağlanmak:

    cqlsh <%server ip%>

komutunu terminal üzerinde çalıştırarak cassandra sunucusuna bağlanıyoruz. Sonrasında oluşuturlmuş veritabanı var ise;

  

    USE <%keysapce ismi%>;

  

  

komutunu girerek bu database’in scope’una giriyoruz.

  

    describe tables

komutunu yazarsak bize Keyspace ve altındaki tabloları gösterir.

  
**USE** komutu ile Database scope’una girdikten sonra

  

    Select *from table

  

şeklinde sorgu atıp sonuç alabiliriz.

# Murmurhash(Cryptography):

[http://www.wiki-zero.co/index.php?q=aHR0cHM6Ly9lbi53aWtpcGVkaWEub3JnL3dpa2kvTXVybXVySGFzaA](http://www.wiki-zero.co/index.php?q=aHR0cHM6Ly9lbi53aWtpcGVkaWEub3JnL3dpa2kvTXVybXVySGFzaA)

# Tombstone 

Veri silindiğinde her data için tombstone oluşturur,, bunun tamamen diskten uçurulma süresi **cassandra.yml** içerisinde **gc_grace_seconds** parametresidir.

  

# Compact

Veriler silindikten sonra disk işlemlerini hızlandırmak için sıkıştırma işlemi yapılır, normalde bunu otomatik olarak kendisi yapar, eğer manuel olarak yapılmak istenirse. **nodetool compact** komutu kullanılarak yapılabilir.

  

  

# Flush

Cassandra tablolarının şifrelenmiş dosyalarını disk üzerinde görmek için flush işlemi yapılır. Cassandra folder’ı altında bulunan tablo adının dosyasına uygulanır.

  

    bin/nodetool flush <%key_space_name%> <%table_name%>

  

bu komutu kullandıktan sonra db dosyalarına erişebilir oluyoruz ve detaylarını görmek için Cassandra plugini olan **sstable2tojson** komutuyla db’yi json olarak yazdırabiliriz.

  

**Örnek olarak:**

  

    bin/sstable2tojson /var/lib/cassandra/data/<%keyspace_name%>/<%table_name%>/<%keyspace_name%>-<%table_name%>-jb-3-Data.db

  

Sonrasında bunu sıkıştırmak için

  

    bin/nodetool compact <%key_space_name%> <%table_name%>

  

# T.T.L. (Time to live):

Eklenilen değerin ne kadar veritabanı üzerinde duracağını set eden bir özellik. Örnek olarak geçici bir token oluşturup bunu sonradan kill etmek istiyoruz TTL değeri set ederek istediğimiz sürede kayıtı tabloda tutabiliriz.

  

# INSERT 

  

    INSERT INTO cycling.calendar (race_id, race_name, race_start_date, race_end_date) VALUES (200, 'placeholder', '2015-05-27', '2015-05-27') USING TTL 30;

# UPDATE 

  

    UPDATE cycling.calendar USING TTL 259200
    SET race_name = 'Tour de France - Stage 12' WHERE race_id = 200 AND race_start_date = '2015-05-27' AND race_end_date = '2015-05-27’;

**TTL(Time to live) değerini sıfırlamak için;**

    UPDATE cycling.calendar USING TTL 0

    SET race_name = 'Tour de France - Stage 12'
	WHERE race_id = 200 AND race_start_date = '2015-05-27' AND race_end_date = '2015-05-27';

# Role Management:

**Kullanıcıları Listele**

    list roles;

**Yeni kullanıcı eklemek için;**

`create role <%Role name%> with password=‘<%password%>’ and LOGIN =true and superuser=true`

**Kullanıcı Silmek için;**

    drop role <%Role Name%>

# MODEL OLUŞTURMA
Model dediğimiz bir tablodur aslında tablo alanlarını kurgularken düşündüğümüz soyut yapı denilebilir. Cassandra da bu Static table ya da Entity Table denilebilir.

    CREATE TABLE documents (
	   docid uuid,
	   categoryid uuid,
	   docname varchar,
	   longdocdetail varchar,
	   created_date timestamp
	   PRIMARY KEY (docid)
	);

Cassandra veritabanında Primary Key diğer ilişkisel veritabanlarının aksine daha farklı bir yapıdadır. Tek bir alan Primary Key olarak seçilmişse bu aslında bir Partition Key'dir. Partition Key Cassandra için benzersiz alan olarak tanımlanabilir. Eğer mimari yapısı dağıtık olarak ayarlanmışsa aynı zamanda verileri hangi Node'lara dağıtacağını bu alan üzerinden belirler. Cassandra yazacağı verinin tek bir Node'da olmasını kesinlikle sağlar sebebi ise veriyi aradığımızda tüm Node'lara bakmak yerine kendi yazdığı Node üzerinden veriyi getirir bu da performans sağlar.

![Cassandra Nodes](https://image.ibb.co/fjrM18/node_example.png)

**COMPLEX PRIMARY KEY**
Cassandra'da Dynamic Table olarak bir tablo tipi bulunmaktadır. İlk **PRİMARY KEY** örneğinde verdiğimiz **docid** alanı tekil birincil anahtardı birden fazla alanı **PRİMARY KEY** yapabiliriz.
Aşağıdaki örnekte birden fazla alan birincil anahtar olarak verilmiştir, bu özellik kullanıcıya göre dökümanın hızlıca gösterilebilmesini sağlar. Burada **Partition Key** aslında **docid** alanıdır diğer iki alan olan **created_date** ve **editorid** **Clustering Column** görevi görmektedir. **Clustering Column**'lar verinin partition sırasını belirler, **Partition Key** ise hangi node altında tutulacağını belirler.

      CREATE TABLE documents (
	   docid uuid,
	   categoryid uuid,
	   editorid,
	   docname varchar,
	   longdocdetail varchar,
	   created_date timestamp
	   PRIMARY KEY (docid, created_date, editorid)
	);
	
**Partition Key** --> docid
**First Clustering Column** --> created_date
**Second Clustering Column** --> editorid

Veriler yukarıdaki oluşumda aşağıdaki gibi node'lara yazılırlar. **Default** olarak **ORDER** değeri **ASC'**dir.

    editorid 1 created_date 1 docid1 docname
    editorid 2 created_date 2 docid2 docname
    editorid 3 created_date 3 docid3 docname
    editorid 4 created_date 3 docid4 docname

Clustering Column'lar partition üzerinde sıralama yapma amacı ile kullanıldığı için bu alanları CLUSTERING ORDER BY parametresi ile kullanmak daha doğru olacaktır.

       CREATE TABLE documents (
	   docid uuid,
	   categoryid uuid,
	   editorid,
	   docname varchar,
	   longdocdetail varchar,
	   created_date timestamp
	   PRIMARY KEY (docid, created_date, editorid)
	   WITH CLUSTERING ORDER BY (created_date DESC, docid ASC)
	);
Yukarıdaki şekilde tabloyu değiştirdiğimizde ise sıralama aşağıdaki gibi değişecektir.

    editorid 1 created_date 4 docid1 docname
    editorid 2 created_date 3 docid2 docname
    editorid 3 created_date 2 docid3 docname
    editorid 4 created_date 1 docid4 docname

Sizin ihtiyacınıza göre bu parametreleri etkin kullanmak çok önemlidir, Cassandra kullanım amacı performans odaklı işlemler olduğu için dikkat edilecek en ufak nokta çok hızlı veri almanıza / yazmanıza sebebiyet verecektir.


# INDEX İŞLEMLERİ

-   **Partition Key**  verinin hangi nodelarda tutulacağını belirler
-   **Clustering Key** partition key içerisindeki verinin hangi sırada tutulacağını belirler
-   **Primary Key** (Compound/Simple)Partition key ve Clustering Key oluşturmak için kullanılır
-   **Composite/Compound Key** parition key oluşturmak için birden fazla kolon kullanmak zorunda olduğumuz durumlarda geçerlidir.

Sources: sorsorgula.com, datastax, csql-engine, udemy.com
