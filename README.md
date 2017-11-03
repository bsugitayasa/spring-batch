# Spring Batch #

Roadmap

* Overview Spring Batch
* Build from scratch
* Live Demo

## Batch in Spring ##

### Overview Spring Batch ###

* Spring Batch merupakan framework yang mengakomudasi proses secara batch, dengan mengeksekusi serangkaian `job`
* Job terdiri dari serangkaian step
* Step terdiri dari proses
    * Read
    * Process
    * Write
* Step juga dapat terdiri dari single operation atau istilah dalam spring batch disebut `Tasklet`
* Workflow dasar spring batch sebagai berikut

![workflow-spring-batch](img/batch-diagram.jpg)

### Beberapa Istilah Spring Batch ###

1. Job : mendeskripsikan sebuah pekerjaan, misal nya membaca file *.csv dan kemudian menyimpan ke dalam database. Sebuah Job juga merupakan container dari satu atau beberapa kumpulan Step
2. Step : merupakan bagian independent yang mengandung semua informasi terkait kontrol dari proses batch. Sebuah Job terdiri dari satu atau lebih Step, misalnya Step 1 untuk membaca file *.csv dan Step 2 adalah untuk mapping nilai file kedalam database
3. JobInstance : sebuah instance yang sedang berjalan dari sebuah Job yang telah ditetapkan dalam konfigurasi. Contoh Job untuk membaca file, artinya kita memiliki 1 JobInstance dengan kata lain 1 Job yang sedang berjalan sama dengan 1 JobInstance
4. JobParameters : sekumpulan parameter yang digunakan oleh JobInstance
5. JobExecution : merupakan hasil dari menjalankan setiap JobInstance. Misal dalam membaca file *.csv, Job pertama mengalami kegagalan sedangkan berhasil setelah Job dijalankan ulang. Maka kita akan mempunyai 1 JobInstance dan memiliki 2 JobExecution (1 yang gagal, 1 yang berhasil)
6. StepExecution : memiliki arti yang sama layaknya JobExecution namun lebih mempresentasikan hasil dari sebuah Step
7. JobRepository : sebuah persistent store untuk menyimpan semua informasi meta-data sebuah Job. Repository diperlukan untuk mengetahui state dari sebuah Job, apakah mengalami kegagalan atau tidak dalam proses nya. Secara default informasi disimpan pada memory, namun dapat diseting untuk disimpan dalam database
8. JobLauncher : object yang memungkinkan kita untuk memulai sebuah Job menggunakan JobRepository untuk mendapatkan JobExecution yang valid
9. Tasklet : situasi dimana kita tidak memiliki input dan pengolahan dari output, misalnya ingin menjalankan/memanggil sebuah store procedure
10. ItemReader : sebuah abstract class yang digunakan untuk menggambarkan sebuah object yang memungkinkan kita untuk membaca sebuah object yang ingin diproses, misalnya membaca file atau membaca data dari database
11. ItemProcessor : sebuah abstract class yang digunakan untuk menghandel bisnis logic dati data yang dihasilkan oleh ItemReader sebelum dilempar ke ItemWriter
12. ItemWriter : sebuah abstract class yang digunakan untuk menulis hasil akhir dari proses batch


### Enable Batch ###

* Simply adding the ```@EnableCaching```

### Caching With Annotations ###

* ```@Cacheable```
* ```@CachePut```
* ```@CacheEvict```
* ```@CacheConfig```

## Build dan Run ##

### Persiapan Database ###

* Buat database baru pada MySQL (misal : spring_cahce_demo)


### Spring Boot Initializr ###

1. Browse ke [https://start.spring.io/] (https://start.spring.io/)

2. Lengkapi Project Metadata

    Group
    
    ```
    com.sheringsession.balicamp.springcache    
    ```

    Artifact
    
    ```
    demo
    ```
   
    Dependencies
    
    ```
    Web, JPA, Cache, MySQL (Optional tergantung DB yang digunakan)
    ```
   
3. Generate Project

4. Download dan import pada IDE masing-masing


### Build with Maven ###

1. Pastikan `pom.xml` terdapat dependency berikut
    
    ```
    <dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId> 
			<scope>runtime</scope> 
        </dependency>
    </dependencies>
    ```

2. `applicatioin.properties` untuk konfigurasi database source, hibernate dll

    ```
    spring.datasource.url=jdbc:mysql://localhost/spring_cahce_demo
    spring.datasource.username=disesuaikan
    spring.datasource.password=disesuaikan
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
    
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.show-sql=true
    spring.jpa.properties.hibernate.format_sql=true
    
    spring.datasource.max-wait=40000
    spring.datasource.test-on-borrow=true
    spring.datasource.validation-query=SELECT 1
    
    spring.jackson.serialization.indent-output=true
    ```


### Build with your IDE ###

1. Buat Enity misal Product

    ```
    @SuppressWarnings("serial")
    @Data @Entity @Table
    public class Product implements Serializable{
        @Id @GeneratedValue(generator="uuid") @GenericGenerator(name="uuid", strategy= "uuid2")
        private String id;
        private String code;
        private String name;
        private BigDecimal amount;
        private Boolean outOfStock;
    }
    ```

2. Jalankan aplikasi

    ```
    mvn clean spring-boot:run
    ```
    Cek tabel Product otomatis akan terbentuk dalam database

3. Buat DAO `ProductDao` 

    ```
    public interface ProductDao extends PagingAndSortingRepository<Product, String>{
        public Product findByName(String name);
        public Product findByCode(String code);
    }
    ```

4. Buat Service `ProductService`

    ```
    public class ProductService {
        private static final Logger LOG = LoggerFactory.getLogger(ProductService.class);
        @Autowired private ProductDao productDao;
    
        public Product findProductByName(String name) {
            LOG.info("### Call service product by name {}", name);
            return productDao.findByName(name);
        }
        
        public Product findProductByCode(String code) {
            LOG.info("### Call service product by code {}", code);
            return productDao.findByCode(code);
        }
        
        public Product updateProduct(String id, String name) {
            LOG.info("### Call service put product id {}, name {}", id, name);
            Product p = productDao.findOne(id);
            p.setName(name);
            productDao.save(p);
            return p;
        }
        
        public void deleteProduct(String name) {
            LOG.info("### Call service evict product name {}", name);
        }
    }
    ```

4. Buat Controller `ProductController`

    ```
    @RestController @RequestMapping("/spring-cache/product")
    public class ProductController {
        private static final Logger LOG = LoggerFactory.getLogger(ProductController.class);
        @Autowired private ProductService productService;

        @GetMapping("/getByName/{name}")
        public Product getDataProductByName(@PathVariable String name) {
            LOG.info("## product controller getByName called here!!");
            return productService.findProductByName(name);
        }

        @GetMapping("/getByCode/{code}")
        public Product getDataProductByCode(@PathVariable String code) {
            LOG.info("## product controller getByCode called here!!");
            return productService.findProductByCode(code);
        }

        @GetMapping("/updateProduct/{id}/{name}")
        public Product updateDataProduct(@PathVariable String id, @PathVariable String name) {
            LOG.info("## product controller put product called here!!");
            return productService.updateProduct(id, name); 
        }

        @GetMapping("/deleteProduct/name/{name}")
        public String deleteDataProduct(@PathVariable String name) {
            LOG.info("## product controller delete product called here!!");
            productService.deleteProduct(name);
            return "Delete Product Cache berhasil !!!";
        }
    }
    ```
    
5. Jalankan menggunakan maven dan browse ke [http://localhost:8080/]