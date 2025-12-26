# 🚀 НАСТРОЙКА ВЫДЕЛЕННОГО СЕРВЕРА 5.249.160.54

## ✅ ЧТО НУЖНО СДЕЛАТЬ:

### ШАГ 1: Подключитесь к серверу
```bash
ssh root@5.249.160.54
# или ваш пользователь
```

### ШАГ 2: Проверьте Node.js
```bash
node --version
npm --version
```
Если нет - установите:
```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# CentOS/RHEL
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs
```

### ШАГ 3: Скопируйте backend файлы
```bash
# Создайте директорию
mkdir -p /opt/lone-star-chat/backend
cd /opt/lone-star-chat/backend

# Скопируйте файлы с вашего Mac (используйте scp или rsync)
# scp -r /Users/svetanaborvinskaia/Desktop/Lone\ Star\ Chat/backend/* root@5.249.160.54:/opt/lone-star-chat/backend/
```

### ШАГ 4: Установите зависимости
```bash
cd /opt/lone-star-chat/backend
npm install
```

### ШАГ 5: Запустите backend
```bash
PORT=666 node server-chat-current.js &
```

### ШАГ 6: Проверьте работу
```bash
curl -X POST http://localhost:666/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin","password":"admin"}'
```

### ШАГ 7: Настройте Nginx (если нужно)
```bash
# Создайте config
sudo nano /etc/nginx/sites-available/ypilo.com

# Вставьте:
server {
    listen 80;
    server_name ypilo.com;

    location / {
        proxy_pass http://localhost:666;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Включите сайт
sudo ln -s /etc/nginx/sites-available/ypilo.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### ШАГ 8: Настройте автозапуск backend
```bash
# Создайте service
sudo nano /etc/systemd/system/lone-star-chat.service

# Вставьте:
[Unit]
Description=Lone Star Chat Backend
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/lone-star-chat/backend
ExecStart=/usr/bin/node server-chat-current.js
Environment=PORT=666
Restart=always

[Install]
WantedBy=multi-user.target

# Включите и запустите
sudo systemctl enable lone-star-chat
sudo systemctl start lone-star-chat
```

### ШАГ 9: Проверьте с внешнего IP
```bash
curl -X POST http://5.249.160.54/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin","password":"admin"}'
```

## 🎉 ГОТОВО!

После настройки ypilo.com будет работать с мобильным интернетом!</content>
<parameter name="filePath">/Users/svetanaborvinskaia/Desktop/Lone Star Chat/НАСТРОЙКА_ВЫДЕЛЕННОГО_СЕРВЕРА.md