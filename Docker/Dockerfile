FROM alpine:3.8

RUN apk add --no-cache python3 && \
    python3 -m ensurepip && \
    rm -r /usr/lib/python*/ensurepip && \
    pip3 install --upgrade pip setuptools && \
    if [ ! -e /usr/bin/pip ]; then ln -s pip3 /usr/bin/pip ; fi && \
    if [[ ! -e /usr/bin/python ]]; then ln -sf /usr/bin/python3 /usr/bin/python; fi && \
    rm -r /root/.cache

# install packages and cleanup
RUN apk update && \
apk add --no-cache git python openssh terraform bzr  && \
apk add --no-cache python &&\
apk add --no-cache openssh &&\
apk add --no-cache terraform &&\
apk add --no-cache bzr &&\
apk add --no-cache go &&\
apk add --no-cache musl-dev &&\
apk add --no-cache bash &&\
apk add --no-cache py-pip

ENV ANSIBLE_VERSION=2.6.2

RUN set -xe \
&& echo "****** Install system dependencies ******" \
&& apk add --no-cache --progress python py-pip openssl \
ca-certificates git openssh sshpass \
&& apk --update add --virtual build-dependencies \
python-dev libffi-dev openssl-dev build-base \
\
&& echo "****** Install ansible and python dependencies ******" \
&& pip install --upgrade pip \
&& pip install ansible==${ANSIBLE_VERSION} boto boto3 \
\
&& echo "****** Remove unused system librabies ******" \
&& apk del build-dependencies \
&& rm -rf /var/cache/apk/*

RUN set -xe \
&& mkdir -p /etc/ansible \
&& echo -e "[local]\nlocalhost ansible_connection=local" > \
/etc/ansible/hosts

RUN rm -rf /tmp/* && \
rm -rf /var/cache/apk/*

# install runtastic opennebula terraform provider
RUN go get -v github.com/runtastic/terraform-provider-opennebula && \
    go install -v github.com/runtastic/terraform-provider-opennebula

# set opennebula driver in .terraformrc
RUN echo -e 'providers {\n\
 opennebula = "/root/go/bin/terraform-provider-opennebula"\n\
 }'\
>> /root/.terraformrc

# change from sh to bash
RUN rm /bin/sh && ln -sf /bin/bash /bin/sh

# WORKDIR
WORKDIR /code

# clone kubespray repo
RUN git clone --branch v2.8.0 https://github.com/kubernetes-incubator/kubespray.git

# install requirements
RUN pip install -r kubespray/requirements.txt
# create ssh-keypair
RUN ssh-keygen -t rsa -f /code/id_rsa_kubespray -q -P ""
