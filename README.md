# Laboratorium 8

W ramach zadania utworzono klaster Minikube z CNI Calico oraz 3 węzłami roboczymi A, B, C.
Następnie wdrożono:
- Deployment `frontend` (nginx, 3 repliki) przypięty do węzła A
- Deployment `backend` (nginx, 1 replika) przypięty do węzła B
- Pod `my-sql` (mysql) przypięty do węzła C
Utworzono usługi:
- `svc-frontend` typu NodePort
- `svc-backend` oraz `svc-mysql` typu ClusterIP
Opracowano NetworkPolicy, która:
- blokuje połączenia z Podów `frontend` do `my-sql`
- zezwala na połączenia z Podów `backend` do `my-sql` tylko na porcie 3306/TCP

Pliki w repo
- `01-frontend-deploy.yaml` – Deployment `frontend`
- `02-backend-deploy.yaml` – Deployment `backend` 
- `03-mysql-pod.yaml` – Pod `my-sql` 
- `04-frontend-svc-nodeport.yaml` – Service `svc-frontend` 
- `05-backend-svc-clusterip.yaml` – Service `svc-backend` 
- `06-mysql-svc-clusterip.yaml` – Service `svc-mysql` 
- `07-netpol-mysql.yaml` – NetworkPolicy dla `my-sql` 
- `08-test-frontend.yaml` – Pod testowy (etykieta `app=frontend`) do sprawdzenia blokady
- `09-test-backend.yaml` – Pod testowy (etykieta `app=backend`) do sprawdzenia dostępu

# 1) Start klastra (Calico + 4 nody: master + A,B,C)
- `minikube delete`
- `minikube start --driver=docker --container-runtime=docker --network-plugin=cni --cni=calico --nodes=4`

# 2) Label węzłów A/B/C
- `kubectl get nodes`
- `kubectl label node minikube-m02 lab8-role=A --overwrite`
- `kubectl label node minikube-m03 lab8-role=B --overwrite`
- `kubectl label node minikube-m04 lab8-role=C --overwrite`

# 3) Wdrożenie zasobów
- `kubectl apply -f 01-frontend-deploy.yaml`
- `kubectl apply -f 02-backend-deploy.yaml`
- `kubectl apply -f 03-mysql-pod.yaml`
- `kubectl apply -f 04-frontend-svc-nodeport.yaml`
- `kubectl apply -f 05-backend-svc-clusterip.yaml`
- `kubectl apply -f 06-mysql-svc-clusterip.yaml`
- `kubectl apply -f 07-netpol-mysql.yaml`
- `kubectl apply -f 08-test-frontend.yaml`
- `kubectl apply -f 09-test-backend.yaml`

# W celu zweryfikowania działania polityki sieciowej uruchomiłem dwa pody testowe:
- test-frontend z etykietą app=frontend`
- test-backend z etykietą app=backend.
Następnie z każdego poda wykonałem test połączenia TCP do usługi svc-mysql na porcie 3306, używając poleceń:
  `kubectl exec -it test-frontend -- nc -zvw2 svc-mysql 3306` oraz
  `kubectl exec -it test-backend  -- nc -zvw2 svc-mysql 3306`
# Wyniki:
- z poda test-frontend połączenie zostało zablokowane - nc zakończył się komunikatem o przekroczeniu czasu („timed out”)
- z poda test-backend połączenie zostało zestawione poprawnie – nc zwrócił komunikat succeeded

# [Potwierdzenie działania na zdjęciach 1.png-9.png w folderze "zrzuty ekranu"]






