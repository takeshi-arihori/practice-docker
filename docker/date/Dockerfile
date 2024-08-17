FROM ubuntu:20.04

RUN apt update
RUN apt install -y vim

COPY docker/date/.vimrc /root/.vimrc

CMD date +"%Y/%m/%d %H:%M:%S ( UTC )"
