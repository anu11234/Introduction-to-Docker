# Introduction to Docker || **GSP055**

**Command:**

```bash
export LOCATION=$(gcloud config get-value compute/zone | cut -d- -f1-2)
export PROJECT_ID=$(gcloud config get-value project)
gcloud config set project $PROJECT_ID
```

```bash
# Task 1: Test basic docker functionality
docker run hello-world

# Task 2: Create project files and build initial image v0.1
mkdir -p ~/test && cd ~/test

cat << 'EOF' > Dockerfile
FROM node:lts
WORKDIR /app
ADD . /app
EXPOSE 80
CMD ["node", "app.js"]
EOF

cat << 'EOF' > app.js
const http = require("http");
const hostname = "0.0.0.0";
const port = 80;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader("Content-Type", "text/plain");
  res.end("Welcome to Cloud\n");
});

server.listen(port, hostname, () => {
  console.log("Server running at http://%s:%s/", hostname, port);
});

process.on("SIGINT", function () {
  console.log("Caught interrupt signal and will exit");
  process.exit();
});
EOF

docker build -t node-app:0.1 .
docker build -t node-app:0.2 .
```

```bash
# Set explicit variables for the lab environment
export LOCATION="us-east1"
export PROJECT_ID=$(gcloud config get-value project)

# 1. Configure Docker authentication
gcloud auth configure-docker ${LOCATION}-docker.pkg.dev --quiet

# 2. Create the Artifact Registry repository
gcloud artifacts repositories create my-repository \
    --repository-format=docker \
    --location=${LOCATION} \
    --description="Docker repository" \
    --quiet || true

# 3. Tag the image for Artifact Registry
docker tag node-app:0.2 ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/my-repository/node-app:0.2

# 4. Push the image to Artifact Registry
docker push ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/my-repository/node-app:0.2
```
