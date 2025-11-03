# Native Node.js Proxy Server

Proxy server untuk mengakses HTTP/HTTPS API, download file, dan WebSocket connection melalui proxy. Support SSL certificate bypass dan file besar.

## Quick Start

```bash
# Install dependencies
npm install

# Start server
npm run dev
```

Server berjalan di `http://localhost:3000`

---

## 🔗 HTTP/HTTPS Proxy

**Cara kerja:** Proxy akan meneruskan request Anda ke target URL dan mengembalikan response-nya.

### Basic Usage

**GET Request:**
```bash
curl "http://localhost:3000/proxy?url=https://api.example.com/users"
```

**POST Request dengan JSON body:**
```bash
curl -X POST "http://localhost:3000/proxy?url=https://api.example.com/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"secret123"}'
```

**PUT Request:**
```bash
curl -X PUT "http://localhost:3000/proxy?url=https://api.example.com/users/1" \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com"}'
```

**DELETE Request:**
```bash
curl -X DELETE "http://localhost:3000/proxy?url=https://api.example.com/users/1"
```

**Dengan Headers Kustom:**
```bash
curl "http://localhost:3000/proxy?url=https://api.example.com/data" \
  -H "Authorization: Bearer your-token-here" \
  -H "X-API-Key: your-api-key"
```

### Real Example (API yang Anda pakai)

```bash
# Login ke API
curl -X POST "http://localhost:3000/proxy?url=https://119.13.101.169/api/auth/admin/v1/en/client/auth" \
  -H "Content-Type: application/json" \
  -d '{"id": 1, "secret": "1873hjasdASIqwe97aASawf98th"}'

# Response: {"access_token":"eyJ...","expired_at":"2025-11-13T20:24:17.451794952+08:00","type":"Bearer"}

# Gunakan token untuk GET data
curl "http://localhost:3000/proxy?url=https://119.13.101.169/api/master/admin/v1/en/country/list" \
  -H "Authorization: Bearer eyJ..."
```

---

## 💾 File Download

**Cara kerja:** Download file dengan streaming (memory efficient), support file besar dan resume download.

### Basic Download

```bash
# Download file ke direktori current
curl -O "http://localhost:3000/download?url=https://example.com/largefile.zip"

# Download dengan nama kustom
curl -o my-file.zip "http://localhost:3000/download?url=https://example.com/file.zip"
```

### Resume Download (Lanjutkan Download)

```bash
# Jika download terputus, lanjutkan dengan -C -
curl -C - -O "http://localhost:3000/download?url=https://example.com/largefile.zip"
```

### Download dengan Range Request

```bash
# Download 1MB pertama saja
curl -H "Range: bytes=0-1048575" "http://localhost:3000/download?url=https://example.com/largefile.zip"

# Download dari byte 1MB ke 2MB
curl -H "Range: bytes=1048576-2097151" "http://localhost:3000/download?url=https://example.com/largefile.zip"
```

### Example Download File Besar

```bash
# Download 100MB file (akan menampilkan progress)
curl -O "http://localhost:3000/download?url=https://httpbin.org/bytes/104857600"

# Output di terminal:
# Download progress: 25.50% (2.5 MB/s)
# Download progress: 50.00% (2.8 MB/s)
# Download completed: 100.00 MB in 35.67 seconds
```

---

## 🔌 WebSocket Proxy

**Cara kerja:** Proxy akan membuat jalur WebSocket antara client Anda dan target WebSocket server.

### Cara Pakai WebSocket Proxy

**Step 1:** Dapatkan info WebSocket target
```bash
curl "http://localhost:3000/ws-proxy?url=wss://echo.websocket.org"
```

**Step 2:** Connect ke WebSocket proxy (dengan JavaScript)

```html
<script>
// Connect ke proxy server
const ws = new WebSocket("ws://localhost:3000/ws-proxy");

ws.onopen = function(event) {
  console.log("Connected to proxy");

  // Kirim target WebSocket URL
  ws.send(JSON.stringify({
    target: "wss://echo.websocket.org"
  }));
};

ws.onmessage = function(event) {
  const data = JSON.parse(event.data);

  if (data.type === 'connected') {
    console.log("Connected to target WebSocket:", data.target);
    // Sekarang bisa kirim pesan ke target
    ws.send("Hello from proxy!");
  } else {
    // Pesan dari target WebSocket
    console.log("Received:", data);
  }
};

ws.onerror = function(error) {
  console.error("WebSocket error:", error);
};
</script>
```

### Example: WebSocket Echo Test

```javascript
const ws = new WebSocket("ws://localhost:3000/ws-proxy");

ws.onopen = () => {
  // Connect ke echo server
  ws.send(JSON.stringify({ target: "wss://echo.websocket.org" }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type !== 'connected') {
    console.log("Echo response:", data); // Akan muncul pesan yang sama
  }
};

// Kirim pesan
setTimeout(() => {
  ws.send("Hello WebSocket!");
}, 1000);
```

---

## 🏥 Health Check & Monitoring

### Cek Server Status
```bash
curl "http://localhost:3000/health"
```

Response:
```json
{
  "status": "OK",
  "timestamp": "2025-11-03T12:30:00.000Z",
  "version": "1.0.0",
  "proxyType": "native-nodejs",
  "endpoints": {
    "proxy": "/proxy?url=<target-url>",
    "download": "/download?url=<file-url>",
    "websocket": "/ws-proxy?url=<ws-url>",
    "health": "/health"
  },
  "features": ["HTTP/HTTPS proxy", "File download streaming", "WebSocket proxy"]
}
```

### Lihat Semua Endpoint
```bash
curl "http://localhost:3000/"
```

---

## 🛠️ Usage Examples (Bahasa Pemrograman)

### JavaScript/Node.js

```javascript
// HTTP/HTTPS Proxy
const proxyUrl = `http://localhost:3000/proxy?url=${encodeURIComponent('https://api.example.com/data')}`;

fetch(proxyUrl)
  .then(res => res.json())
  .then(data => console.log(data));

// POST dengan data
const postData = {
  username: "testuser",
  password: "testpass"
};

fetch(`http://localhost:3000/proxy?url=${encodeURIComponent('https://api.example.com/login')}`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(postData)
})
.then(res => res.json())
.then(data => console.log(data));

// WebSocket
const ws = new WebSocket("ws://localhost:3000/ws-proxy");
ws.onopen = () => {
  ws.send(JSON.stringify({ target: "wss://echo.websocket.org" }));
};
ws.onmessage = (event) => console.log(JSON.parse(event.data));
```

### Python

```python
import requests
import urllib.parse
import websocket

# HTTP/HTTPS Proxy
api_url = "https://api.example.com/data"
proxy_url = f"http://localhost:3000/proxy?url={urllib.parse.quote(api_url)}"
response = requests.get(proxy_url)
print(response.json())

# POST request
login_data = {"username": "test", "password": "test"}
login_url = "https://api.example.com/login"
proxy_login = f"http://localhost:3000/proxy?url={urllib.parse.quote(login_url)}"
response = requests.post(proxy_login, json=login_data)
print(response.json())

# File download
file_url = "https://example.com/largefile.zip"
download_url = f"http://localhost:3000/download?url={urllib.parse.quote(file_url)}"
response = requests.get(download_url, stream=True)
with open("downloaded_file.zip", "wb") as f:
    for chunk in response.iter_content(chunk_size=8192):
        f.write(chunk)
```

## Configuration

Environment variables di `.env`:

```env
PORT=3000
NODE_ENV=development
CORS_ORIGIN=*
```

## Deployment

### PM2
```bash
pm2 start src/index.js --name "proxy"
```

### Docker
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

## Examples

### JavaScript
```javascript
const proxyUrl = `http://localhost:3000/proxy?url=${encodeURIComponent(apiUrl)}`;
fetch(proxyUrl).then(res => res.json()).then(console.log);
```

### Python
```python
import requests, urllib.parse
proxy_url = f'http://localhost:3000/proxy?url={urllib.parse.quote(api_url)}'
response = requests.get(proxyUrl)
```

## Troubleshooting

**Masalah Umum:**

1. **404 Not Found** - URL target tidak ada → Cek URL langsung
2. **Connection timeout** - Server tidak reachable → Cek koneksi/network
3. **SSL Certificate Error** - SSL kadaluarsa → Proxy bypass otomatis
4. **Invalid URL format** - URL salah format → Gunakan URL encoding

**Tips Debug:**
- Lihat server logs untuk detail error
- Test URL langsung sebelum diproxy
- Gunakan browser dev tools

## License

MIT License

## Project Structure

```
proxy/
├── src/
│   └── index.js              # Main proxy server
├── package.json              # Dependencies
├── README.md                 # Documentation
├── .env                      # Environment variables
└── .env.example              # Environment template
```