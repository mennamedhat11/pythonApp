# Python App

تطبيق Flask بسيط، متعمل له CI/CD باستخدام Jenkins وDocker.

## الـ Endpoints
- `GET /` : رسالة ترحيب
- `GET /health` : فحص الحالة

## التشغيل المحلي
```bash
pip install -r requirements.txt
python app.py
```

## التشغيل بـ Docker
```bash
docker build -t python-app .
docker run -d -p 5000:5000 python-app
```
افتحي `http://localhost:5000`

## الـ CI/CD (Jenkins)
الـ `Jenkinsfile` بيعمل الخطوات دي:
1. Clone للريبو
2. Build لـ Docker image
3. Push للـ image على Docker Hub
4. Run للكونتينر
5. Smoke test على `/health`
