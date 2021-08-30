# Jalankan SCA Tool Snyk Scanner pada Pipeline
Snyk : tool scanner untuk menganalisa tingkat keamanan code  
> Snyk Language Support : https://support.snyk.io/hc/en-us/articles/360020352437-Language-support-summary
## Instalasi dan konfigurasi Snyk Plugin di Jenkins
1. Signup Snyk dengan menggunakan akun github
2. Install Snyk Plugin di jenkins : Manage Jenkins -> Manage Plugins -> Available -> Snyk Security Plugin -> Install and Restart
3. Manage Jenkins -> Global Tools Configuration -> Snyk Installation -> Add Snyk  
![image](/images/Snyk-1.png)
4. Nama bebas dan unik
5. Pastikan Install Automatically dicentang
6. Update Policy Interval adalah interval snyk untuk memeriksa update
7. Save
8. Ambil auth token dari akun snyk : https://app.snyk.io/account > General > Auth Token > Click to View
9. Buat kredensial snyk di Jenkins
![image](/images/Snyk-2.png)
10. Save 
## Integrasi Snyk dengan Pipeline
1. Dashboard > book-pipeline > Pipeline Syntax > Generate Syntax
![image](/images/Snyk-3.png)
2. Pada syntax tersebut severity vulnerabilitynya adalah medium or above. Maka jika ada vulnerability dengan severity medium atau high pipeline akan dibreak
3. Jenkinsfile : https://raw.githubusercontent.com/takatux/books/master/jenkins/Jenkinsfile-book-snyk
4. Output Pipeline Monitor
![image](/images/Snyk-4.png)
5. Snyk juga akan mengirim notif via email
![image](/images/Snyk-5.png)
6. Snyk Report juga akan muncul di Jenkins Dashboard  
![image](/images/Snyk-6.png)
7. Vulnerabilities issue dapat dilihat di dashboard snyk  
![image](/images/Snyk-7.png)
8. Snyk juga akan mengirimkan Pull Request untuk memperbaiki code
![image](/images/Snyk-8.png)

## Open Pull Request dari Snyk
jika github terintegrasi dengan snyk, snyk dapat memberi rekomendasi perubahan kode dengan mengirim pull request
1. Snyk Dashboard
2. Vulnerable Projects > klik Fix this vulnerabilities
3. Create Pull Request
4. Open Fix PR  
![image](/images/Snyk-9.png)
5. Cek repo github  
![image](/images/Snyk-10.png)
