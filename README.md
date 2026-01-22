# Práctica de Despliegue de Aplicaciones  
## Flask + Gunicorn
**2º DAW**

---

## 📌 Introducción

En esta práctica se ha realizado el despliegue completo de una aplicación web desarrollada con **Flask**, utilizando un entorno de producción basado en **Gunicorn** como servidor WSGI y **Nginx** como proxy inverso.  

## 🔹 1. Instalación de prerrequisitos

En primer lugar se actualizó el sistema e instalaron los paquetes necesarios para poder trabajar con Python y desplegar la aplicación.

### Instalación de pip

sudo apt-get update
sudo apt-get install -y python3-pip
python3 --version

<img width="467" height="48" alt="requisito python3" src="https://github.com/user-attachments/assets/22889bc6-3b96-4b93-a844-c99aac7b3188" />

Instalación de Pipenv : Pipenv se instaló sin privilegios de administrador para evitar crear entornos virtuales en directorios del sistema.


pip3 install pipenv
pipenv --version


<img width="467" height="48" alt="requisito pipenv" src="https://github.com/user-attachments/assets/69e1e663-c642-403f-9d55-bff79e4c1330" />


Instalación de python-dotenv : Este paquete se utiliza para cargar variables de entorno desde el archivo .env.


pip3 install python-dotenv
pip3 --version

<img width="467" height="48" alt="requisito pip3" src="https://github.com/user-attachments/assets/7c485401-fa62-4e65-8688-fae5754d6c4a" />



gunicorn --version

<img width="467" height="48" alt="requisito gunicorn" src="https://github.com/user-attachments/assets/ce0067c0-d322-40d3-b126-f69cf2afc186" />


🔹 2. Creación del directorio del proyecto
Siguiendo la metodología indicada en la práctica, el proyecto se ubicó en el directorio /var/www.


sudo mkdir -p /var/www/app
sudo chown -R vagrant:www-data /var/www/app
sudo chmod -R 775 /var/www/app
Estos permisos son necesarios para que Nginx pueda acceder correctamente a la aplicación.


<img width="448" height="60" alt="3 Creacion de carpeta y ajuste de permisos" src="https://github.com/user-attachments/assets/c37688a0-1ef7-4f06-885f-1758e30bcf89" />


🔹 3. Variables de entorno
Dentro del directorio del proyecto se creó el archivo .env, que contiene las variables necesarias para Flask en producción.



FLASK_APP=wsgi.py
FLASK_ENV=production


<img width="564" height="70" alt="4 Modificación del archivo  env del proyecto" src="https://github.com/user-attachments/assets/77f9fcd0-8e46-431a-8f58-bcc37ec8c45a" />



🔹 4. Creación del entorno virtual e instalación de dependencias
Desde el directorio del proyecto se creó y activó el entorno virtual con Pipenv.


pipenv shell
pipenv install flask gunicorn



<img width="922" height="291" alt="5 Creación del entorno virtual" src="https://github.com/user-attachments/assets/d43711c7-e8f9-43e0-85c0-538551489a4c" />


Instalación de dependencias del entorno virtual 


<img width="663" height="291" alt="6 Instalación dependencias en entorno virutal" src="https://github.com/user-attachments/assets/7817f17b-aac3-461b-bcd6-30fbbb2e5b44" />


🔹 5. Desarrollo de la aplicación Flask (PoC)
Se creó una aplicación Flask mínima como prueba de concepto.


Archivo application.p


<img width="569" height="195" alt="7 Contenido de application py " src="https://github.com/user-attachments/assets/afb4fcf6-aa92-4198-8c94-0e6d3e8273d6" />

    
Archivo wsgi.py


<img width="562" height="111" alt="8 Contenido de wsgi py " src="https://github.com/user-attachments/assets/0d8f113d-6859-4365-af59-56f0df7d4e6f" />



🔹 6. Pruebas en local
En primer lugar deberemos de poner el puerto 5000 tanto en guest como en el host de nuestra maquina virtual


<img width="509" height="27" alt="10 adicion de redireccion de puertos para pruebas " src="https://github.com/user-attachments/assets/b6c9044d-afcf-4669-a347-aabcd8070402" />


Servidor de desarrollo de Flask


flask run --host 0.0.0.0

<img width="856" height="162" alt="9 Prueba manual " src="https://github.com/user-attachments/assets/8bd04e85-393e-483d-a142-e0fc2b5f964c" />

<img width="314" height="132" alt="11 legada Flask" src="https://github.com/user-attachments/assets/f90043ac-6c6b-4541-85a0-58a5a44b62ab" />


Prueba con Gunicorn


gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app


<img width="653" height="147" alt="12 prueba gunicorn" src="https://github.com/user-attachments/assets/7949fc42-73cc-424d-a7b7-b44d48425caa" />


🔹 7. Creación del servicio systemd
Se creó un servicio para que Gunicorn se ejecute automáticamente como un servicio del sistema.

Después se recargó systemd y se inició el servicio:

sudo systemctl daemon-reload
sudo systemctl enable flask_app
sudo systemctl start flask_app


<img width="860" height="73" alt="16 preparacion" src="https://github.com/user-attachments/assets/9590e682-2366-4f5f-8eae-db7bb1ae9060" />


Comprobamos el estado de nuestra flask_app

<img width="762" height="170" alt="17 estado app flask" src="https://github.com/user-attachments/assets/45eea63e-f520-4351-96f6-8c7ff140c5e4" />


🔹 8. Configuración de Nginx
Se configuró Nginx como proxy inverso para servir la aplicación a través de Gunicorn.

Archivo de configuración:

/etc/nginx/sites-available/app.conf


<img width="739" height="257" alt="18 Contenido app conf " src="https://github.com/user-attachments/assets/5087be86-8fb7-48ea-baa7-22d0ef28dda8" />

Se activó el sitio y se reinició Nginx:



sudo ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx


<img width="866" height="111" alt="19 Creacion enlace simbolico y verificacion estado nginc" src="https://github.com/user-attachments/assets/cc7ed1fc-073f-4776-a5d0-de568568f0e8" />




🔹 9. Configuración del archivo hosts
Para poder acceder mediante un nombre de dominio, se modificó el archivo hosts del sistema anfitrión.


192.168.33.10 app.izv www.app.izv


🔹 10. Comprobación final
Se accedió desde el navegador a:


http://app.izv
Mostrando correctamente la aplicación desplegada.


<img width="407" height="112" alt="20 app izv " src="https://github.com/user-attachments/assets/667c90dd-3f49-4c2a-8d91-9b55c3d52f01" />


🔹 11. Tarea de ampliación
Como tarea de ampliación se repitió todo el proceso completo con una aplicación real obtenida del siguiente repositorio:

👉 https://github.com/Azure-Samples/msdocs-python-flask-webapp-quickstart

Durante esta parte se realizaron los mismos pasos:

Clonado del repositorio y permisos


<img width="838" height="179" alt="21 TA clone y permisos" src="https://github.com/user-attachments/assets/4b86e7d7-ff71-41eb-8e9d-c70f732d56a4" />

Modificación del archivo .env


<img width="558" height="83" alt="22 TA archivo  env " src="https://github.com/user-attachments/assets/7b5903d5-8071-490d-8aec-59ab25e0468d" />


Creación del entorno virtual


<img width="1169" height="218" alt="23 TA Creacion del entorno virtual" src="https://github.com/user-attachments/assets/19e05918-2ea7-4376-9254-118f22cadcdf" />


Instalación de dependencias desde requirements.txt


<img width="1040" height="145" alt="24 Instalación de dependencias del proyecto" src="https://github.com/user-attachments/assets/df75c460-4eb8-46c8-9f8e-5e8074223ec2" />

Instalación de dependencias de Guicorn


<img width="962" height="217" alt="25 Instalacion de dependencias de gunicorn" src="https://github.com/user-attachments/assets/035ca8f0-83e3-4037-bc31-e849afa3ac70" />

Pruebas con Flask y Gunicorn


<img width="1117" height="431" alt="25 1 Prueba flask y gunicorn" src="https://github.com/user-attachments/assets/daa4a6e3-f5ea-4ba4-8b6d-761bb56f2de5" />

Prueba flask 


<img width="1238" height="575" alt="26 TA prueba flask" src="https://github.com/user-attachments/assets/f2b28163-27f6-43ea-8621-99213a623b7c" />

Prueba Guicorn


<img width="1183" height="567" alt="27 TA Prueba gunicorn" src="https://github.com/user-attachments/assets/3d77dc50-4441-48b3-b110-e1618d1204ad" />


