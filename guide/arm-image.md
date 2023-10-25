# Build image on x86 to run on ARM Microshift (Python)
- Build base image on ARM in Python starterapp repo:
```
FROM ubi8/python-38:1-129
ENV PORT 5000
EXPOSE 5000
WORKDIR /usr/src/app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

#COPY . .

#ENTRYPOINT ["python"]
#CMD ["wsgi.py"]
```
- Build app into base image on x86 (e.g. OCP):
```
FROM quay.io/nexus6/starter-app-python-base:latest

COPY . .

ENTRYPOINT ["python"]
CMD ["wsgi.py"]
```
