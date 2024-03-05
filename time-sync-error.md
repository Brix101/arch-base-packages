sudo vim /etc/systemd/timesyncd.conf

set NTP=time.google.com

If the Fallback_NTP line is commented out (prefixed with #), uncomment it.

Save

sudo systemctl restart systemd-timesyncd
