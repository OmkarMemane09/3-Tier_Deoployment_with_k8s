1️⃣ Update backend `application.properties` → add RDS details  
2️⃣ Build & push backend Docker image → Docker Hub  
3️⃣ Update `backend.yaml` → set new image  
4️⃣ Apply `backend.yaml` + `backend-svc.yaml`  
5️⃣ Copy backend LB URL → paste into `frontend/.env`  
6️⃣ Build & push frontend image → update `frontend.yaml` → apply  
