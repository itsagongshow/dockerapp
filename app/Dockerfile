# Pull Image
FROM python:3.8-alpine

# Copy Files
COPY ./ /app/

# Working Dir
WORKDIR /app

# Install Required Apps
RUN apk add --update alpine-sdk

RUN apk update \ 
    && apk add python3-dev gcc libc-dev libevent-dev libffi-dev


# Install Requirements
RUN pip install -r requirements.txt

# Start
COPY . /app

EXPOSE 5000
CMD ["python", "userapi.py" ]
