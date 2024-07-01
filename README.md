Achmad Khosyi' Assajjad Ramandanta

5025211007

Progjar (E)

# Time-Server-With-TCP

Buatlah sebuah program time server dengan ketentuan sebagai berikut :
- Membuka port di port 45000 dengan transport TCP
- Server harus dapat melayani request yang concurrent, gunakan contoh kode multithreading berikut
```
from socket import *
import socket
import threading
import logging
import time
import sys

class ProcessTheClient(threading.Thread):
	def __init__(self,connection,address):
		self.connection = connection
		self.address = address
		threading.Thread.__init__(self)

	def run(self):
		while True:
			data = self.connection.recv(32)
			if data:
				self.connection.sendall(data)
			else:
				break
		self.connection.close()

class Server(threading.Thread):
	def __init__(self):
		self.the_clients = []
		self.my_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		threading.Thread.__init__(self)

	def run(self):
		self.my_socket.bind(('0.0.0.0',8889))
		self.my_socket.listen(1)
		while True:
			self.connection, self.client_address = self.my_socket.accept()
			logging.warning(f"connection from {self.client_address}")
			
			clt = ProcessTheClient(self.connection, self.client_address)
			clt.start()
			self.the_clients.append(clt)
	

def main():
	svr = Server()
	svr.start()

if __name__=="__main__":
	main()
```
- Ketentuan request yang dilayani :
  - Diawali dengan string “TIME dan diakhiri dengan karakter 13 dan karakter 10”
  - Setiap request dapat diakhiri dengan string “QUIT” yang diakhiri dengan karakter 13 dan 10 
- Server akan merespon dengan jam dengan ketentuan
  - Dalam bentuk string (UTF-8)
  - Diawali dengan “JAM<spasi><jam>”
  - <jam> berisikan info jam dalam format “hh:mm:ss” dan diakhiri dengan karakter 13 dan karakter 10

## Testing
Untuk menjalankan program, jalankan terlebih dahulu server_thread.py dengan command
```
python3 server_thread.py
```

Lalu pada terminal client, menggunakan netcat tulis perintah
```
nc 172.16.16.101 45000
```
lalu ketik perintah `TIME`, `command random`, dan `QUIT`.

## Kesimpulan
Ketika program dijalankan dengan perintah `python3 server_thread.py`, server akan dimulai pada port 45000 dan menunggu koneksi masuk. Output pada konsol server akan mencatat setiap langkah proses, termasuk waktu server dimulai, koneksi dari klien, dan data yang diterima dari klien. Sebagai contoh, log menunjukkan bahwa server menerima perintah `TIME` dari klien dan mengirimkan kembali waktu saat ini, mencatat perintah tidak dikenal `tes`, dan akhirnya menerima perintah `QUIT` yang menyebabkan penghentian koneksi. 

Di sisi klien, menggunakan `nc 172.16.16.101 45000`, perintah `TIME` menghasilkan respon `JAM 00:20:42`, perintah tidak dikenal tes menghasilkan respon `WARNING: Invalid Command!`, dan perintah `QUIT` menghentikan program dengan pesan "The program will stop and exit". Program ini menunjukkan bagaimana server multithreaded menangani berbagai perintah dari klien secara konkuren, memastikan setiap perintah diproses dan ditanggapi dengan tepat.
