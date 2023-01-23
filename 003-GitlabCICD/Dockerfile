# docker run -p 8080:8080 --name color-web ghcr.io/devops2021/devopsdaydayup:colorweb
# docker run -p 8080:8080 --name color-web briansu2004/udemy-devops-9projects-free:colorweb

FROM python:3.8-alpine

RUN mkdir /app

COPY . /app

WORKDIR /app

RUN pip install -r requirements.txt

EXPOSE 8080

CMD ["python", "app.py"]
