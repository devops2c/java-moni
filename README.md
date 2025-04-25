# java-moni
tester prometheus et grafana

avant stresse
![image](https://github.com/user-attachments/assets/74af8c03-7f61-4037-8b3d-affb43e3aae3)

stresse:
wrk -t5 -c40000 -d400s http://localhost:8080/
-t = threads
-c = connexions simultanées
-d = durée

![image](https://github.com/user-attachments/assets/d0b29eb5-bc99-4cff-bafb-63d65d9b7060)

![image](https://github.com/user-attachments/assets/77a3864c-e4cf-4da4-998d-a3c2e998b265)

