# [Alert Rules](https://prometheus.ingress.k8s.memhamwan.net/rules#APRS)

## APRXNotRunning

This alert means that the aprx service is not correctly running. It may have exited for some reason. The proper next step is to start the service, which will make the alert resolve:

```
sudo systemctl start aprx # Start the APRX service
```

If this does not work, escalate in [#netops-chat](https://discord.com/channels/956750974876254218/956752860983463976)

# General Problems

## If you believe the TNC is hung, restart it with the following command:

```
sudo systemctl stop aprx # Stop the APRX service
sudo pitnc_setparams /dev/ttyS0 0 15 2 # Tell the TNC to reset
sudo pitnc_getparams /dev/ttyS0 0 # Confirm that it is responsive
sudo systemctl start aprx # Start the APRX service
```

Then, confirm that the TTY port is opened by looking at the log:
```
tail /var/log/aprx/aprx.log
```

Finally, check for the received RF signal by watching rf receipt in real time:
```
tail -f /var/log/aprx/aprx-rf.log
```
Exit using control+c