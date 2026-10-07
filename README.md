# Docker-Project
Red Hat Linux is used to create and test the Docker project. GitHub Actions automatically runs lint, testing, Docker build and Docker Hub push after every Git push. Before production deployment, a reviewer gives manual approval.
mkdir myproject
cd myproject
touch app.py requirements.txt Dockerfile
mkdir -p .github/workflows
vi app.py
print("Hello CI/CD")
vi requirements.txt
pytest
flake8

vi Dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]

docker build -t myapp .
docker images
docker run myapp
Hello CI/CD
vi .github/workflows/ci-cd.yml
Simple pipeline:
name: CI-CD
on:
  push:
    branches:
      - main
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install flake8
        run: pip install flake8
      - name: Lint
        run: flake8 app.py
  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Test
        run: python app.py
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker
        run: docker build -t myapp .
  push:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Login Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
      - name: Build
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/myapp:latest .
      - name: Push
        run: docker push ${{ secrets.DOCKER_USERNAME }}/myapp:latest
  deploy:
    needs: push
    runs-on: ubuntu-latest
    environment:
      name: production
    steps:
      - name: Deploy
        run: echo "Production deployment successful"
