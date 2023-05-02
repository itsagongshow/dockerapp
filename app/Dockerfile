# start by pulling the python image
FROM python:3.8-alpine

# copy the requirements file into the image
COPY ./ /app/

# switch working directory
WORKDIR /app

RUN apk add --update alpine-sdk

RUN apk update \ 
    && apk add python3-dev gcc libc-dev libevent-dev libffi-dev


# install the dependencies and packages in the requirements file
RUN pip install -r requirements.txt

# copy every content from the local file to the image
COPY . /app

EXPOSE 5000
CMD ["python", "userapi.py" ]