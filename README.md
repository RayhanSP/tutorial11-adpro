# Refleksi Modul Tutorial 11: Deployment on Kubernetes

**Nama:** Rayhan Syahdira Putra
**NPM:** 2306275903
**Kelas:** Pemrograman Lanjut B

---

## Tutorial: Hello Minikube - Refleksi

### 1. Perbandingan Log Aplikasi Sebelum dan Sesudah Diekspos sebagai Service

Sebelum aplikasi diekspos sebagai Service, log yang tercatat pada Pod `hello-node` umumnya terbatas pada indikasi *startup* server internal, seperti inisialisasi server HTTP pada *port* 8080 dan server UDP pada *port* 8081. Log ini mengonfirmasi bahwa aplikasi di dalam Pod telah berhasil berjalan dan siap menerima koneksi, namun belum ada indikasi aktivitas eksternal karena Pod belum terekspos ke luar *cluster* Kubernetes.

Setelah aplikasi diekspos sebagai Service menggunakan `minikube service hello-node` dan diakses melalui *browser*, log aplikasi akan menunjukkan peningkatan yang signifikan. Setiap kali aplikasi diakses atau halaman di-*refresh*, entri log baru akan muncul, mengindikasikan adanya permintaan HTTP yang masuk dan diproses oleh aplikasi di dalam Pod. Fenomena ini membuktikan bahwa Service berhasil meneruskan lalu lintas eksternal ke Pod yang berjalan, dan jumlah log memang akan meningkat seiring dengan frekuensi akses terhadap aplikasi, mencerminkan interaksi yang terjadi pada lapisan aplikasi.

### 2. Tujuan Opsi `-n` pada Perintah `kubectl get`

Opsi `-n` (singkatan dari `--namespace`) pada perintah `kubectl get` berfungsi untuk menentukan *namespace* Kubernetes yang menjadi target operasi perintah tersebut. Secara *default*, `kubectl` akan berinteraksi dengan *namespace* `default` jika tidak ada *namespace* lain yang ditentukan, sehingga objek-objek seperti Deployment dan Service `hello-node` yang kita buat tanpa spesifikasi *namespace* akan berada di dalamnya. Opsi ini krusial untuk mengorganisasi dan mengisolasi sumber daya dalam *cluster* Kubernetes, memastikan bahwa operasi `kubectl` hanya memengaruhi objek-objek di *namespace* yang dimaksud.

Ketika perintah `kubectl get pods,services -n kube-system` dijalankan, *output* yang dihasilkan tidak mencantumkan Pod dan Service `hello-node` yang telah dibuat secara eksplisit karena perintah tersebut secara spesifik diarahkan untuk hanya menampilkan objek-objek yang berada di *namespace* `kube-system`. *Namespace* `kube-system` didedikasikan untuk menampung komponen-komponen inti *cluster* Kubernetes itu sendiri, seperti `coredns`, `kube-apiserver`, dan `metrics-server`. Oleh karena objek `hello-node` berada di *namespace* `default`, mereka tidak akan terlihat saat memfilter berdasarkan *namespace* `kube-system`.

---

## Tutorial: Rolling Update & Kubernetes Manifest File - Refleksi

### 1. Perbedaan antara Strategi Deployment Rolling Update dan Recreate

Strategi *Rolling Update* adalah metode *deployment* di Kubernetes yang memungkinkan pembaruan aplikasi tanpa *downtime*. Proses ini dilakukan dengan mengganti Pod-Pod yang sedang berjalan dengan versi baru secara bertahap atau inkremental. Selama *rolling update*, Pod baru akan dijadwalkan pada Node yang memiliki sumber daya yang tersedia, dan setelah semua Pod baru berhasil dimulai, Pod lama akan dihapus secara bertahap, memastikan ketersediaan aplikasi tetap terjaga sepanjang proses pembaruan.

Berbeda dengan *Rolling Update*, strategi *Recreate* (buat ulang) melibatkan penghentian dan penghapusan semua instans aplikasi versi lama secara keseluruhan sebelum memulai instans aplikasi versi baru. Pendekatan ini menyebabkan *downtime* yang signifikan selama proses pembaruan, karena aplikasi tidak akan tersedia untuk pengguna selama transisi dari versi lama ke versi baru. Meskipun lebih sederhana dalam implementasinya karena tidak perlu mengelola transisi lalu lintas atau kompatibilitas *backward* selama pembaruan, strategi *Recreate* umumnya tidak disarankan untuk aplikasi yang membutuhkan ketersediaan tinggi.

### 2. Percobaan Deploy Spring Petclinic REST Menggunakan Strategi Recreate

Untuk menerapkan strategi *Recreate* pada Spring Petclinic REST, pertama-tama saya memastikan tidak ada Deployment `spring-petclinic-rest` yang aktif agar proses pembuatan ulang dapat diuji dengan jelas. Pendekatan yang paling langsung adalah dengan memodifikasi manifest Deployment yang sudah ada atau membuat manifest baru yang secara eksplisit mendefinisikan strategi *Recreate*. Ini dapat dilakukan dengan menambahkan atau mengubah bagian `strategy` di dalam spesifikasi Deployment YAML.

Setelah manifest disiapkan, saya akan menghapus Deployment lama (jika ada) dan menerapkan manifest baru yang menggunakan strategi *Recreate* melalui perintah `kubectl apply -f [nama_file_manifest.yaml]`. Selama proses ini, saya akan memantau status Pod menggunakan `kubectl get pods` dan *event* *cluster* dengan `kubectl get events`. Observasi yang diharapkan adalah Pod-Pod lama akan terminasi sepenuhnya sebelum Pod-Pod versi baru memulai, menunjukkan adanya *downtime* singkat selama transisi pembaruan.

### 3. Penyiapan File Manifest Berbeda untuk Strategi Recreate

Untuk mengeksekusi strategi *Recreate*, manifest Deployment perlu dimodifikasi dengan menambahkan atau menyesuaikan bagian `spec.strategy`. Berikut adalah contoh bagian yang relevan dalam `deployment.yaml` yang akan diubah:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-petclinic-rest
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-petclinic-rest
  strategy: # Tambahkan atau modifikasi bagian ini
    type: Recreate
  template:
    metadata:
      labels:
        app: spring-petclinic-rest
    spec:
      containers:
      - name: spring-petclinic-rest
        image: docker.io/springcommunity/spring-petclinic-rest:3.2.1 # Versi terbaru
        ports:
        - containerPort: 9966
```

Dengan penambahan strategy.type: Recreate, Kubernetes akan diinstruksikan untuk menghentikan semua Pod yang ada sebelum memulai Pod baru dengan image yang diperbarui. File service.yaml umumnya tidak perlu diubah karena strategi deployment hanya memengaruhi bagaimana Pod diatur, bukan cara Service mengeksposnya.

### 4. Manfaat Penggunaan File Manifest Kubernetes
Penggunaan file manifest Kubernetes (YAML) memiliki manfaat signifikan dibandingkan dengan deployment manual menggunakan perintah kubectl secara langsung. Pertama, file manifest menyediakan single source of truth atau "Infrastructure as Code" yang jelas dan dapat di-version control. Ini memungkinkan perubahan deployment dilacak dengan mudah, kolaborasi tim yang lebih baik, dan kemampuan untuk mengembalikan konfigurasi ke keadaan sebelumnya jika terjadi masalah, yang sangat sulit dilakukan jika hanya mengandalkan perintah manual.

Kedua, file manifest memfasilitasi otomatisasi dan reproduktifitas deployment. Dengan satu file YAML, seluruh konfigurasi aplikasi, termasuk Deployment, Service, dan sumber daya lainnya, dapat diterapkan ke cluster mana pun menggunakan perintah kubectl apply -f. Ini sangat efisien untuk deployment berulang di lingkungan pengembangan, staging, dan produksi, mengurangi potensi kesalahan manusia dan memastikan konsistensi lingkungan.

## File Manifest Kubernetes

Berikut adalah file manifest Kubernetes yang diekspor dari tutorial. File-file ini ditempatkan di direktori yang sama dengan `README.md` ini.

* `deployment.yaml`
* `service.yaml`

