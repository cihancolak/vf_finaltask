1. Setup and Version Control

   
a. Web Uygulaması Oluşturma

Basit bir Flask (Python) uygulaması olusturdum:

app.py:

```

from flask import Flask, jsonify, request

app = Flask(__name__)

# In-memory database
books = []

# Route to get all books
@app.route('/books', methods=['GET'])
def get_books():
    return jsonify(books), 200

# Route to add a new book
@app.route('/books', methods=['POST'])
def add_book():
    new_book = request.get_json()
    books.append(new_book)
    return jsonify(new_book), 201

# Route to get a book by ID
@app.route('/books/<int:book_id>', methods=['GET'])
def get_book(book_id):
    book = next((book for book in books if book['id'] == book_id), None)
    if book:
        return jsonify(book), 200
    else:
        return jsonify({'error': 'Book not found'}), 404

# Route to update a book by ID
@app.route('/books/<int:book_id>', methods=['PUT'])
def update_book(book_id):
    book = next((book for book in books if book['id'] == book_id), None)
    if book:
        data = request.get_json()
        book.update(data)
        return jsonify(book), 200
    else:
        return jsonify({'error': 'Book not found'}), 404

# Route to delete a book by ID
@app.route('/books/<int:book_id>', methods=['DELETE'])
def delete_book(book_id):
    global books
    books = [book for book in books if book['id'] != book_id]
    return '', 204

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)



```

2. Dockerization

Dockerfile

```
FROM python:3.8-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]


```
Docker konteynerinin temelini oluşturmak için "python:3.8-slim-buster" görüntüsünü kullanır. Bu, Python 3.8 içeren ve Debian Buster tabanlı hafif bir görüntüdür.

konteyner içinde /app dizinine geçiş yapar ve bu dizini çalışma dizini olarak ayarlar. Bundan sonraki tüm komutlar bu dizin içinde çalıştırılacaktır.

COPY requirements.txt requirements.txt: Bu satır, host sistemde bulunan requirements.txt dosyasını konteyner içindeki /app dizinine kopyalar.
RUN pip install -r requirements.txt: Bu satır, requirements.txt dosyasındaki tüm bağımlılıkları pip kullanarak yükler. Bu dosya, projede kullanılan Python kütüphanelerini listeler.

Bu satır, host sistemdeki tüm dosyaları ve dizinleri (Dockerfile hariç) konteyner içindeki mevcut çalışma dizinine (/app) kopyalar.

Bu satır, konteyner başlatıldığında çalıştırılacak komutu belirtir. Bu örnekte, app.py dosyasını Python ile çalıştırır. app.py, Flask uygulamanızı başlatan ana Python dosyasıdır.

Bu adımlar, Docker konteynerinde Flask uygulamanızı çalıştırmak için gerekli olan ortamı hazırlar. 



Dockerfile olusturduktan sonra my-flask-app isimli image"i olusturalim ve run edelim.


```
docker build -t my-flask-app .
docker run -p 5000:5000 my-flask-app
```

Ayrica image imizi Dockerhub registry olarak bulabiliriz: 


```
https://hub.docker.com/r/cihancolak/my-flask-app
```


minikube start komutumuzla minikube olusturabiliriz.

Deployment.yaml dosyamiz:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: cihancolak/my-flask-app
        ports:
        - containerPort: 5000
      imagePullSecrets:
      - name: myregistrykey
 
```

Bu dosya, Kubernetes Deployment nesnesini tanımlayan bir YAML dosyasıdır. 
Deployment, uygulamanızın belirtilen sayıda pod içinde çalışmasını sağlar ve gerektiğinde bu pod'ları yeniden başlatır veya günceller.

apiVersion: apps/v1: Bu, Deployment nesnesinin kullanacağı API sürümünü belirtir.
kind: Deployment: Bu, oluşturulacak nesnenin türünü belirtir. Burada, bir Deployment nesnesi oluşturulacağını belirtir.

name: flask-app: Deployment nesnesine verilen isimdir. Bu, Kubernetes içinde bu Deployment nesnesini tanımlamak için kullanılır.

Bu kısım, Deployment nesnesinin nasıl davranacağını ve hangi özelliklere sahip olacağını belirler.

replicas: 2: Bu, kaç tane pod'un çalıştırılacağını belirtir. Bu örnekte, iki kopya çalıştırılacaktır.

matchLabels: Bu, pod'ların eşleşmesi gereken etiketleri belirtir. Bu durumda, app: flask-app etiketiyle eşleşen pod'lar seçilecektir.

metadata:

    labels: Bu etiketler, oluşturulan pod'lara uygulanır. app: flask-app etiketiyle işaretlenirler.

spec:

    containers: Bu kısım, pod içinde çalışacak konteynerleri tanımlar.
        name: flask-app: Konteynerin adı.
        image: cihancolak/my-flask-app: Kullanılacak Docker imajı. Bu imaj Docker Hub veya özel bir Docker registry'de bulunabilir.
        ports: Konteynerin hangi port üzerinden erişilebilir olacağını belirtir.
            containerPort: 5000: Konteynerin 5000 numaralı portunu açar.
    imagePullSecrets: Özel bir Docker registry'den imaj çekerken kullanılacak gizli anahtarlar.
        name: myregistrykey: Bu, Kubernetes Secret nesnesinin adıdır. Bu secret, Docker registry'den imaj çekmek için gerekli kimlik bilgilerini içerir.



Service.yaml dosyamiz 


```

apiVersion: v1
kind: Service
metadata:
  name: flask-app-service
spec:
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer





```

bu YAML dosyası bir Kubernetes Service nesnesini tanımlıyor. Service nesneleri, uygulamalar arasında iletişimi sağlar ve dış dünyadan erişim sağlar.

apiVersion: v1: Bu, Kubernetes API'nin kullanılacak sürümünü belirtir.
kind: Service: Bu, oluşturulacak nesnenin türünü belirtir. Burada, bir Service nesnesi oluşturulacaktır.

name: flask-app-service: Service nesnesine verilen isimdir. Bu isim, Kubernetes içinde bu Service nesnesini tanımlamak için kullanılır.


selector: Bu, Service'nin hangi pod'ları hedef alacağını belirtir. Burada, app: flask-app etiketine sahip pod'lar hedef alınacaktır.

ports: Bu kısım, Service üzerinde hangi portların açılacağını belirtir.

    protocol: TCP: Bu, iletişim protokolünü belirtir. Burada TCP kullanılıyor.
    port: 80: Bu, dış dünyadan erişilecek olan port numarasını belirtir.
    targetPort: 5000: Bu, Service'nin yönlendireceği pod'ların hangi portunu dinleyeceğini belirtir.

type: Bu, Service'nin hangi türde olacağını belirtir. LoadBalancer, dış dünyadan erişilebilir olması için bir Load Balancer atar.

Uygulamamizi kubernetes cluster'a deploy etmek icin : 

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```


erisebilirligi kontrol etmek icin de bu komutu run edebiliriz :
```
minikube service flask-app-service
```

Jenkis kurulumunu yaptiktan sonra :


![alt text](https://i.imgur.com/9dcnVyG.png)

![alt text](https://i.imgur.com/bMlY1MT.png)

![alt text](https://i.imgur.com/cUbNCQ8.png)


Jenkinsfile

```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    // Docker imajını oluştur
                    def dockerImage = docker.build('my-flask-app')
                }
            }
        }
        stage('Test') {
            steps {
                // Testlerin çalıştırılması
                sh 'echo "Running tests..."'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Kubernetes YAML dosyalarının uygulanması
                    sh 'kubectl apply -f deployment.yaml 
                    sh 'kubectl apply -f service.yaml 
                }
            }
        }
    }
    post {
        always {
            // Çalışma alanının temizlenmesi
            cleanWs()
        }
    }
}



```
agent any: Bu, Pipeline'ın Jenkins üzerinde herhangi bir ajan üzerinde çalışacağını belirtir. Yani, Jenkins sunucusunda uygun bir ajan seçilecektir.


Build: Docker imajının oluşturulması için adımlar içerir.
Test: Testlerin çalıştırılması için adımlar içerir.
Deploy: Kubernetes YAML dosyalarının uygulanması için adımlar içerir.

Bu bölüm, Pipeline tamamlandığında gerçekleştirilecek son adımları belirtir. cleanWs() fonksiyonu, çalışma alanının temizlenmesini sağlar.
Pipeline'ı Çalıştırma

Pipeline'ı çalıştırmak için Jenkins arayüzünden veya Jenkins CLI kullanarak bir job oluşturmalısınız. Pipeline'ı Jenkins arayüzü üzerinden oluşturmak için aşağıdaki adımları izleyebiliriz:

    Jenkins arayüzünde "Yeni İş" (New Item) seçeneğine tikladim.
    İsim verin ve "Pipeline" seçeneğini sectik.
    Oluşturduğunuz job'un yapılandırma sayfasına gidin.
    "Pipeline" sekmesine gidin.
    "Definition" altında "Pipeline script" seçeneğini seçin.
    Pipeline kodunu yapıştırın.
    Değişiklikleri kaydedin ve Pipeline'ı başlatın.



Build outputlari:

![alt text]( https://i.imgur.com/VPFezpE.png)


![alt text](https://i.imgur.com/Xbb1OnB.png )



Prometheus icin gerekli ayarlamalari yapalim

app.py icin bu guncellemeyi yaptim

```
from flask import Flask, jsonify, request
from prometheus_client import Counter, generate_latest, CONTENT_TYPE_LATEST

app = Flask(__name__)

# Prometheus metrics
REQUEST_COUNT = Counter('request_count', 'Total number of requests')

@app.route('/')
def home():
    REQUEST_COUNT.inc()
    return "Hello, World!"

@app.route('/metrics')
def metrics():
    return generate_latest(), 200, {'Content-Type': CONTENT_TYPE_LATEST}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)


```


Bu bolumde monitor etmek icin prometheus ve metricsleri grafana ile nasil gorsellestirdigimizi gosterecem:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/prometheus
```

```
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
```


```

kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
```

Check this command 

```
kubectl get svc
```


Prometheus setup ve running olduktan sonra 

```

minikube service prometheus-server-ext --url

```




 ![alt text](https://i.imgur.com/1VvPhcz.png )


  ![alt text](https://i.imgur.com/j28Knee.png)


![alt text](https://i.imgur.com/116oVj8.png  )


Grafana ile bağlantı kurulduktan sonra, Prometheus tarafından toplanan metriklerin görselleştirilmesi için Grafana'yı yapılandıracağız.
Bu işlem, Prometheus'tan gelen verilerin Grafana panolarında anlamlı grafikler ve görseller şeklinde sunulmasını sağlayacaktır. 
Bu sayede sistem performansını ve uygulama sağlığını daha etkin bir şekilde izleyebilir ve analiz edebiliriz.


  ![alt text](https://i.imgur.com/32Mf3DZ.png)

 ![alt text](https://i.imgur.com/0KPYFue.png)




 ![alt text](https://i.imgur.com/lgOtaUH.png )


