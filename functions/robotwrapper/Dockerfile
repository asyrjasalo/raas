ARG WATCHDOG_VERSION=0.5.3
ARG PYTHON_VERSION=3.6.8

FROM openfaas/of-watchdog:"$WATCHDOG_VERSION" as watchdog
FROM python:"$PYTHON_VERSION"-alpine

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog
CMD ["fwatchdog"]

RUN apk add --update --no-cache bash tzdata && \
    rm -rf /var/cache/* /tmp/* /var/log/* ~/.cache
RUN cp /usr/share/zoneinfo/Europe/Helsinki /etc/localtime
RUN echo "Europe/Helsinki" > /etc/timezone

COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade -r requirements.txt

COPY rest.robot .
COPY robotwrapper .
ENV fprocess="./robotwrapper"

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1
