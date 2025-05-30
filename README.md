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

Opsi `-n` (singkatan dari `--namespace`) pada perintah `kubectl get` berfungsi untuk menentukan *namespace* Kubernetes yang menjadi target operasi perintah tersebut. Secara *default*, `kubectl` akan berinteraksi dengan *namespace* `default` jika tidak ada *namespace* lain yang ditentukan, sehingga objek-objek seperti Deployment dan Service `hello-node` yang kita buat tanpa spesifikasi *namespace* akan berada di dalamnya. Opsi ini krusial untuk mengorganisasi dan mengisolasi sumber daya dalam *cluster* Kubernetes, memastikan bahwa operasi `kubectl` hanya memengaruhi objek-objinjek di *namespace* yang dimaksud.

Ketika perintah `kubectl get pods,services -n kube-system` dijalankan, *output* yang dihasilkan tidak mencantumkan Pod dan Service `hello-node` yang telah dibuat secara eksplisit karena perintah tersebut secara spesifik diarahkan untuk hanya menampilkan objek-objek yang berada di *namespace* `kube-system`. *Namespace* `kube-system` didedikasikan untuk menampung komponen-komponen inti *cluster* Kubernetes itu sendiri, seperti `coredns`, `kube-apiserver`, dan `metrics-server`. Oleh karena objek `hello-node` berada di *namespace* `default`, mereka tidak akan terlihat saat memfilter berdasarkan *namespace* `kube-system`.

---

## File Manifest Kubernetes

Berikut adalah file manifest Kubernetes yang diekspor dari tutorial. File-file ini ditempatkan di direktori yang sama dengan `README.md` ini.

* `deployment.yaml`
* `service.yaml`