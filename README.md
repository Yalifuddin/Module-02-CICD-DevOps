# Module-01-Coding-Standards

## Refleksi 1

---

### Prinsip Clean Code yang Diterapkan

1. **Separation of Concerns**, saya menerapkannya dengan memisahkan controller, service, dan repository ke dalam package yang berbeda. Pemisahan ini membuat setiap lapisan memiliki tanggung jawab yang jelas dan mencegah pencampuran logika antar layer, sehingga kode lebih mudah dipahami dan dimaintain.

2. **Single Responsibility Principle**, saya memastikan setiap kelas hanya memiliki satu tanggung jawab utama. Controller hanya bertugas menangani request dan response HTTP, service menangani logika bisnis, dan repository menangani penyimpanan serta pengambilan data.

3. **Meaningful Naming**, saya melakukan penamaan kelas, metode, dan variabel secara deskriptif seperti `ProductController`, `ProductService`, `create`, `findAll`, `findById`, `update`, dan `deleteById`. Penamaan ini memungkinkan tujuan dan fungsi kode dapat dipahami tanpa perlu membaca implementasinya secara mendetail.

4. **Don't Repeat Yourself (DRY)**, saya menggunakan Lombok untuk mengurangi boilerplate code seperti getter dan setter. Template Thymeleaf juga memanfaatkan fragment dan reusable components untuk menghindari duplikasi markup HTML.

5. **Consistent Formatting**, saya menerapkan format kode yang konsisten dengan indentasi seragam, penempatan kurung kurawal yang konsisten, dan organisasi import statement yang teratur untuk meningkatkan readability.

---

### Praktik Secure Coding yang Diterapkan

1. **Avoiding SQL Injection Risks**, pada tahap ini repository masih menggunakan in-memory storage sehingga tidak ada interaksi langsung dengan database. Ketika nantinya menggunakan database, akan diterapkan prepared statement atau ORM untuk mencegah SQL injection.

2. **UUID untuk ID Produk**, penggunaan UUID untuk generate ID produk secara otomatis mencegah prediktabilitas ID dan mengurangi risiko serangan enumeration attack dimana attacker mencoba menebak ID produk secara berurutan.

3. **Post/Redirect/Get Pattern**, saya menerapkan pattern ini setelah operasi yang mengubah data seperti create, update, dan delete. Dengan melakukan redirect setelah POST, aplikasi dapat mencegah pengiriman ulang data yang sama ketika pengguna melakukan refresh halaman.

4. **Minimalisir Data Exposure**, view template hanya menampilkan data yang benar-benar dibutuhkan oleh user interface. Hal ini membantu mengurangi risiko kebocoran data sensitif yang tidak perlu.

---

### Kesalahan yang Ditemukan dan Cara Perbaikan

1. **Penghapusan data menggunakan metode GET tidak aman** karena dapat memicu penghapusan tidak disengaja melalui prefetch, web crawler, atau disalahgunakan melalui link berbahaya. Perbaikan yang disarankan adalah menggunakan metode POST atau DELETE dengan perlindungan CSRF melalui Spring Security dan menggunakan form submission untuk delete action.

2. **Validasi input belum diterapkan** pada field seperti `productName` dan `productQuantity`. Field name bisa menerima string kosong atau terlalu panjang, sedangkan quantity bisa menerima nilai negatif. Perbaikan dapat dilakukan dengan menambahkan anotasi validasi Bean Validation seperti `@NotBlank`, `@Size`, dan `@Min` pada model, serta menangani validation error di controller untuk menampilkan pesan error yang informatif.

3. **Penanganan error masih minim** karena ketika data produk tidak ditemukan, aplikasi hanya melakukan redirect tanpa memberikan informasi kepada pengguna. Perbaikannya adalah dengan mengubah return type method menjadi `Optional`, melakukan pengecekan keberadaan data, dan menampilkan pesan error atau halaman khusus agar pengguna memahami apa yang terjadi.

4. **Manajemen ID produk masih lemah** karena ID dihasilkan di repository dengan pengecekan sederhana. Jika ID diberikan secara manual tanpa validasi ketat, bisa terjadi duplikasi atau inkonsistensi. Perbaikannya adalah dengan selalu menghasilkan ID di sisi server menggunakan UUID dan tidak mengizinkan client untuk menentukan ID.

5. **Tipe input quantity kurang tepat** karena masih menggunakan `type="text"` yang memungkinkan input karakter non-numerik. Perbaikan dapat dilakukan dengan mengganti menjadi `type="number"` dengan atribut `min` untuk validasi client-side, namun tetap menambahkan validasi server-side karena client-side validation dapat di-bypass.

---

## Refleksi 2

---

### Unit Test, Code Coverage, dan Kualitas

Setelah menulis unit test, saya merasa lebih percaya diri karena setiap fungsi sudah terdokumentasi dan dapat diverifikasi secara otomatis. Unit test juga berfungsi sebagai safety net ketika melakukan refactoring atau menambah fitur baru, sehingga dapat segera mendeteksi jika ada fungsionalitas yang rusak.

Jumlah unit test dalam sebuah kelas sebaiknya disesuaikan dengan **kompleksitas fungsi dan logika** yang perlu diverifikasi. Prinsip yang saya gunakan adalah **satu test untuk satu skenario**, yang memungkinkan semua edge case dapat terverifikasi dengan baik. Selain itu, fokus pada testing public methods dan behavior, bukan detail implementasi internal.

Untuk class repository misalnya, minimal harus ada test untuk:
- Method `create()`: skenario produk baru, produk dengan ID existing, produk dengan ID null
- Method `findAll()`: list kosong, list dengan beberapa produk
- Method `findById()`: ID valid, ID tidak ditemukan, ID null
- Method `update()`: produk existing, produk tidak ditemukan
- Method `deleteById()`: ID valid, ID tidak ditemukan

Untuk memastikan test sudah cukup, saya mengombinasikan beberapa pendekatan:

1. **Memastikan semua public method dan kondisi penting sudah diuji** dengan fokus pada happy path, edge cases, dan error scenarios.

2. **Menggunakan Code Coverage sebagai panduan**, metrik ini mengukur persentase kode yang dieksekusi saat test berjalan. Ada beberapa jenis coverage: line coverage (persentase baris yang dieksekusi), branch coverage (persentase cabang kondisi yang dieksekusi), method coverage, dan class coverage. Tools seperti JaCoCo, Cobertura, atau IntelliJ IDEA built-in coverage dapat digunakan untuk mengukur coverage. Target yang realistis adalah 80-90% untuk production code.

3. **Menerapkan Mutation Testing** seperti PIT/PITest untuk mengecek kualitas test dengan mengubah kode dan melihat apakah test mendeteksi perubahan tersebut.

**Apakah 100% coverage berarti bebas bug?** Jawabannya adalah **TIDAK**. Alasannya:

- **Coverage tidak sama dengan correctness**. Test bisa mengeksekusi semua baris kode tanpa melakukan assertion yang kuat atau tepat. Misalnya test yang hanya memanggil method tanpa memvalidasi hasilnya.

- **Tidak menguji semua kombinasi input**. Impossible untuk test semua kemungkinan input, sehingga edge cases atau corner cases tertentu bisa terlewat.

- **Tidak mendeteksi logic error**. Bug bisa tersembunyi pada logika bisnis yang kompleks, atau test mungkin mengecek hasil yang salah sehingga bug tetap lolos meskipun test pass.

- **Tidak menguji integration issues**. Unit test mengisolasi class sehingga tidak menguji interaksi antar komponen. Bug bisa muncul ketika komponen digabungkan.

- **Tidak menguji concurrency issues** seperti race conditions atau deadlocks yang hanya muncul pada kondisi multi-threading.

- **Tidak menguji runtime issues** seperti memory leaks, performance problems, atau timeout.

Kesimpulannya, code coverage adalah **alat bantu**, bukan **tujuan akhir**. Yang lebih penting adalah **quality of tests** daripada quantity of coverage. Test harus mencakup berbagai skenario dan dikombinasikan dengan integration tests, functional tests, dan manual testing.

---

### Refleksi Clean Code pada Suite Uji Fungsional Baru

Jika saya membuat suite uji fungsional baru yang sangat mirip dengan `CreateProductFunctionalTest`, kemudian menyalin prosedur setup dan variabel instance yang sama, maka **code quality akan menurun** karena beberapa alasan:

**Clean Code Issues yang Teridentifikasi:**

1. **Duplicate Code (Violation of DRY Principle)**
   - Codingan setup yang sama (annotations, fields, method setup) muncul berulang di banyak test class
   - Perubahan kecil pada setup membutuhkan edit di banyak tempat yang berbeda
   - Maintenance overhead meningkat signifikan seiring bertambahnya jumlah test class

2. **Inisialisasi Berulang**
   - Variabel instance seperti `serverPort`, `testBaseUrl`, dan `baseUrl` dideklarasikan identik di setiap class
   - Method `setupTest()` dengan anotasi `@BeforeEach` ditulis persis sama berkali-kali
   - Annotations seperti `@SpringBootTest` dan `@ExtendWith` diulang di setiap class
   - Risiko inkonsistensi meningkat jika ada yang lupa mengupdate salah satu class

3. **Lack of Abstraction**
   - Tidak ada base class atau shared setup yang mengenkapsulasi common functionality
   - Setiap test class harus mengurus setup sendiri meskipun kebutuhannya identik
   - Helper methods tidak ter-centralized sehingga juga harus diduplikasi

4. **Reduced Maintainability**
   - Sulit untuk maintain karena perubahan konfigurasi test harus direplikasi ke semua test class
   - Error-prone karena mudah lupa update salah satu class saat ada perubahan
   - Testing overhead tinggi: lebih banyak boilerplate daripada actual test logic
   - Developer baru akan kesulitan memahami pola testing karena terlalu banyak repetisi

**Dampak terhadap Code Quality:**

Ya, **code duplication akan mengurangi kualitas kode** karena:
- **Maintainability menurun**: lebih sulit untuk maintain dan update
- **Readability menurun**: developer harus membaca banyak boilerplate yang sama berulang kali
- **Error-prone**: tingginya risiko inconsistency antar test classes
- **Violates SOLID Principles**: khususnya prinsip DRY dan Single Responsibility

**Perbaikan yang Dapat Dilakukan:**

1. **Membuat Base Test Class**
   - Buat abstract class misalnya `BaseFunctionalTest` yang berisi semua common setup
   - Pindahkan annotations, instance variables, dan method `setupTest()` ke base class
   - Deklarasikan instance variables sebagai `protected` agar accessible dari subclass
   - Tambahkan helper methods yang reusable seperti `navigateToProductList()`, `navigateToCreateProduct()`, dan `createProduct()`
   - Setiap test class cukup extends `BaseFunctionalTest` tanpa perlu menulis ulang setup code

2. **Menggunakan Page Object Pattern**
   - Buat class terpisah untuk setiap halaman web (contoh: `ProductListPage`, `CreateProductPage`)
   - Page Object mengenkapsulasi element locators dan interactions untuk halaman tersebut
   - Test class tidak langsung berinteraksi dengan Selenium API, tapi melalui Page Objects
   - Keuntungan: abstraksi yang baik, reusability tinggi, maintainability meningkat, dan readability lebih baik

3. **Menggunakan @BeforeEach atau @BeforeAll secara konsisten**
   - Manfaatkan lifecycle hooks di base class untuk setup yang konsisten
   - Hindari duplikasi setup logic di setiap test method

4. **Membuat Helper atau Test Utility Class**
   - Centralisasi common operations seperti create test data, navigate, atau assertions
   - Misalnya `createTestProducts(count)` untuk membuat sejumlah produk test sekaligus
   - `countProductsInList()` untuk menghitung jumlah produk di list

5. **Test Parameterization**
   - Jika skenario test berbeda hanya pada input dan expected output, gunakan parameterized test
   - Mengurangi jumlah test methods yang repetitive

**Keuntungan dari Refactoring:**

- **DRY Compliance**: tidak ada code duplication
- **Better Maintainability**: perubahan setup hanya di satu tempat
- **Improved Readability**: test lebih fokus pada behavior, bukan boilerplate
- **Reusability**: helper methods dan page objects bisa digunakan ulang
- **Consistency**: setup yang seragam di semua test
- **Easier Testing**: lebih mudah dan cepat menulis test baru


Tes Push Module 2
Tes Workflow Module 2
