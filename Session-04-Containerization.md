# FP04 - Containerization and Configuration Management

## Configuration Management

### Exerciții

1. Instalați Ansible pe mașina `gitlab.fiipractic.lan`.
   ```bash
   dnf install epel-release
   dnf install ansible
   ```

2. Verificați dacă instalarea a avut loc cu succes.
   ```bash
   ansible --version
   ```

3. Creați un folder nou numit `Ansible`, iar în interiorul acestuia un fișier numit `inventory.ini` cu următorul conținut:
   ```ini
   [app]
   app.fiipractic.lan
   [gitlab]
   gitlab.fiipractic.lan
   ```
   Acesta va juca rol de inventar pentru Ansible.

4. Generați o cheie de SSH pe mașina `gitlab.fiipractic.lan` (vedeți instrucțiunile de la prima sesiune).

5. Adăugați cheia publică de SSH de pe mașina `gitlab.fiipractic.lan` pe mașina `app.fiipractic.lan` și pe `gitlab.fiipractic.lan` (vedeți instrucțiunile de la prima sesiune).

6. Verificați dacă vă puteți conecta prin SSH de pe mașina `gitlab.fiipractic.lan` pe mașina `app.fiipractic.lan`.
   ```bash
   ssh root@app.fiipractic.lan
   ```

   > 📝 **Notă:** Pentru a vă putea conecta utilizând adresa `app.fiipractic.lan`, aceasta trebuie să fie trecută în `/etc/hosts` pe mașina `gitlab.fiipractic.lan`. În cazul în care primiți eroarea `No route to host`, verificați conținutul fișierului `/etc/hosts`. Dacă este nevoie, adăugați o intrare pentru `app.fiipractic.lan`:
   > ```bash
   > echo "192.168.100.10 app.fiipractic.lan" >> /etc/hosts
   > ```

7. Verificați dacă puteți ajunge la mașini utilizând Ansible:
   ```bash
   ansible all -i inventory.ini -m ping
   ```

### Ansible Playbook

1. În interiorul directorului creat anterior, descărcați fișierul `playbook.yml` din repo.
   Acest playbook vă va instala docker pe ambele mașini (`app.fiipractic.lan` și `gitlab.fiipractic.lan`).

2. Rulați playbook-ul:
   ```bash
   ansible-playbook -i inventory.ini playbook.yml
   ```

3. Verificați dacă serviciul docker rulează pe mașina `gitlab.fiipractic.lan`:
   ```bash
   systemctl status docker.service
   ```

4. Intrați pe mașina `app.fiipractic.lan` și verificați dacă serviciul docker rulează.

---

## Containerization

> ❗ **Atenție:** Nu este necesar să instalați manual Docker pe mașina `gitlab.fiipractic.lan` dacă ați efectuat pașii din secțiunea anterioară.

### Instalare manuală Docker

1. Adăugați repository-ul pentru Docker pe mașina virtuală:
   ```bash
   sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```

2. Instalați pachetele necesare:
   ```bash
   sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```

3. Porniți serviciul Docker și setați-l pe `enable` pentru a se deschide odată cu mașina virtuală.
   ```bash
   sudo systemctl --now enable docker
   ```

4. Verificați dacă serviciul Docker rulează pe mașina voastră:
   ```bash
   systemctl status docker.service
   ```
   Sau folosiți: `docker images` / `docker ps`.

   > 📝 **Notă:** Aceste comenzi nu vă vor afișa nimic momentan, pentru că nu aveți încă imagini salvate local sau containere care rulează. Vrem doar să vedem că aceste comenzi nu returnează `Command 'docker' not found`.

### Exerciții

> 📝 **Notă:** Următoarele exerciții vor fi efectuate pe mașina `app.fiipractic.lan`.

1. Rulați comanda `docker pull nginx` pentru a descărca ultima imagine de nginx de pe Docker Hub.

2. Verificați dacă imaginea a fost salvată local.

3. Utilizând imaginea descărcată anterior, porniți un container cu numele `container_nginx` și mapați portul 80 de pe container la portul 8080 de pe mașina virtuală.
   ```bash
   docker run --name container_nginx -p 8080:80 -d nginx
   ```

   **Opțiuni:**
   - `--name` → Specifică numele containerului (`container_nginx`).
   - `-p` → Mapează portul 80 (TCP) de pe container la portul 8080 de pe host.
   - `-d` → Rulează în modul *detach* (background).

   > ❗ **Atenție:** Dacă rulați din nou comanda fără să schimbați numele containerului, veți primi:
   > ```
   > docker: Error response from daemon: Conflict. The container name "/container_nginx" is already in use
   > ```
   > Două containere nu pot avea același nume. Redenumiți-l sau ștergeți-l pe cel existent.

   > 📝 **Notă:** Aceeași eroare apare și după `docker stop <container_name>` dacă încercați să porniți unul cu același nume. Containerul încă există — vizibil cu `docker ps -a`. Trebuie șters sau redenumit.

4. Verificați starea containerului cu `docker ps`.
   Intrați în container cu:
   ```bash
   docker exec -it container_nginx /bin/bash
   ```
   Ieșire: `exit`.

5. Verificați că serverul nginx rulează:
   - De pe `app.fiipractic.lan`: `curl -v localhost:8080`
   - Din browser pe gazdă: `http://192.168.100.10:8080`

### Docker volumes

1. Pe `app.fiipractic.lan`, creați un director nou cu numele `Nginx`.

2. Copiați în el `index.html` și `nginx.conf`, împreună cu cheia privată și certificatul SSL de la sesiunea anterioară.

3. Ridicați un container cu numele `container_nginx_2` montând fișierele cu volume Docker:
   ```bash
   docker run -v nginx.conf:/etc/nginx/nginx.conf -v index.html:/var/www/example/index.html [...]
   ```

   > ❗ **Atenție:** De data aceasta mapați portul 443 al containerului la portul 8443 al mașinii virtuale.

### Springboot App - Dockerfile

1. Pe `app.fiipractic.lan`, creați un director nou cu numele `Docker`.

2. Descărcați folderul `springboot` din repo în directorul `Docker`.

3. Tot în acest director, creați un fișier `Dockerfile` cu conținutul din repo.

4. Creați o nouă imagine de Docker:
   ```bash
   docker build -t fiipractic_springboot:1.0 .
   ```

5. Ridicați un container din imaginea creată, mapând portul 8080 al containerului la 8080 al VM-ului.

   > ❗ **Atenție:** Nu puteți mapa același port 8080 la 2 containere diferite. Opriți-l mai întâi pe celălalt.

6. Verificați dacă containerul rulează.
   Dacă nu, vedeți-l cu `docker ps -a`. Dacă starea este `Exited (1)`, verificați motivul cu `docker logs <container_name>`.

7. Accesați pagina Web — `http://app.fiipractic.lan:8080`.

---

## Extra

### Ansible

Creați un playbook de Ansible numit `common.yaml` care să:
- Dezactiveze permanent `firewalld` pe ambele mașini.
- Seteze fusul orar pe `Europe/Bucharest` pe ambele mașini.
- Instaleze Root CA-ul creat în sesiunea anterioară pe ambele mașini.
- Instaleze `htop` pe ambele mașini.
- Seteze `PermitRootLogin` pe `prohibit-password` în `/etc/ssh/sshd_config`.
- Seteze `PasswordAuthentication` pe `no` în `/etc/ssh/sshd_config`.
- Seteze `SELINUX` pe `disabled` în `/etc/sysconfig/selinux`.

### Docker Best Practices

Pornind de la `Dockerfile`-ul furnizat:
- Implementați un **multi-stage build** pentru a reduce dimensiunea imaginii finale.
- Configurați aplicația să ruleze ca **non-root user**.
- Aplicați **limitări de resurse** (CPU și memorie) asupra containerului.

### Docker Compose

1. Pornind de la imaginea din *Springboot App - Dockerfile* și de la exercițiul din *Docker volumes*, scrieți un fișier `docker-compose` care să ridice cele trei containere (2× Springboot + 1× Nginx).

   > ❗ **Atenție:** Nu puteți mapa același port al mașinii virtuale la mai multe containere!

   > ❗ **Atenție:** Doar portul de nginx trebuie mapat pe host — accesul la aplicațiile de springboot se va face exclusiv prin nginx.

2. Pentru fiecare container cu aplicația de springboot, introduceți din `docker-compose` o valoare pentru variabila de environment `MY_VAR`.

### Reverse Proxy

Asemănător cu sesiunea anterioară, configurați nginx ca **reverse proxy** pentru a ajunge la aplicațiile springboot prin 2 rute diferite (sau domenii).

Exemplu:
```
https://app.fiipractic.lan/spring1 -> primul container cu springboot
https://app.fiipractic.lan/spring2 -> al doilea container cu springboot
```

> ❗ **Atenție:** Aplicația de springboot rulează pe portul 8080 în container.

### Load Balancer

Configurați nginx ca un **load balancer**, pentru a distribui traficul către aplicațiile de springboot.

> 📝 **Notă:** În browser dați refresh de câteva ori. Veți observa că ocazional mesajul trimis de aplicație se va schimba în funcție de containerul pe care ajunge request-ul.

> 📝 **Notă:** Puteți înlocui aplicația de springboot cu orice proiect personal.
