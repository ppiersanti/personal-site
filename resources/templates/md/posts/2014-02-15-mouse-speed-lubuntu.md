{:title "Persist Mouse Speed In LXDE"
 :layout :post
 :tags  ["lxde"]
 :toc true
 :draft? false}

## How-To persist mouse speed settings between system restarts...

Edit the file /etc/xdg/lxsession/LXDE/autostart


```bash
# sudo vi /etc/xdg/lxsession/LXDE/autostart
```

Add the following line:


```bash
@xset m <acceleration> <threshold>
```

where acceleration and threshold are your defined values.
