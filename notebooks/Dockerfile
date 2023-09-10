FROM python:3.11

RUN apt-get update && apt-get install -y python3-pip && \
    pip3 install --upgrade pip

WORKDIR /home

COPY ./requirements.txt .

RUN pip3 install --no-cache-dir -r requirements.txt

# Specify the command to run when the container starts
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--allow-root"]
