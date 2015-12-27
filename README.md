tinc-l3
====
Tinc-Repository von Freifunk Stuttgart.
Wer noch kein konfiguriertes tinc hat, kann direkt in /etc/tinc arbeiten, das dazu vorher loeschen, ansonsten TINCBASE anders setzen und bei Dateinamen aufpassen.
TINCBASE=/etc/tinc
if [ -d "$TINCBASE" ]; then
    rm -rf $TINCBASE
fi
git clone git+ssh://git@github.com/freifunk-stuttgart/tinc-l3 $TINCBASE
if [ x"$TINCBASE" != x"/etc/tinc" ]; then
    rsync -rlHpogDtSvx $TINCBASE/. /etc/tinc/
fi
if [ ! -e /etc/tinc/ffsL3/tinc.conf ]; then
    ln -s $TINCBASE/ffsL3/tinc.conf.sample /etc/tinc/ffsL3/tinc.conf
    ln -s $TINCBASE/ffsL3/subnet-up.sample /etc/tinc/ffsL3/subnet-up
    ln -s $TINCBASE/ffsL3/subnet-down.conf.sample /etc/tinc/ffsL3/subnet-down
fi
cd $TINCBASE/ffsL3
tincd -n ffsL3 -K 4096
if [ x"$TINCBASE" != x"/etc/tinc" ]; then
    rsync -rlHpogDtSvx /etc/tinc/ffsL3/hosts/$HOSTNAME  $TINCBASE/ffsL3/hosts/
fi
# Port pro GW verschieden, damit ein GW auch hinter NAT mit UDP funktioniert
cat <<EOF >>hosts/$HOSTNAME
PMTUDiscovery = yes
Digest = sha256
ClampMSS = yes
IndirectData = yes
DirectOnly = no
Address = $HOSTNAME.freifunk-stuttgart.de
Port = 11${HOSTNAME##gw}
EOF

cat <<'EOF' >>/etc/network/interfaces
allow-hotplug ffsL3
auto ffsL3
iface ffsL3 inet manual
    tinc-net ffsL3
    tinc-mlock yes
    tinc-pidfile /var/run/tinc.ffsL3.pid
    up ip l set dev $IFACE up
EOF
