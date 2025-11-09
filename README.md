# mount-size.sh - Partition & Mount Script README

## 📋 Übersicht

Das **mount-size.sh Script** ist ein Tool zur **automatisierten Partitionserstellung mit GPT** auf Linux-Systemen. Es ermöglicht die Erstellung von Partitionen mit spezifischer Größe auf großen Festplatten (>2TB) und das automatische Mounten mit persistenter Speicherung.

**Hauptzweck:** Schnelle und sichere Partitionsverwaltung für große Disks mit flexibler Größenangabe.

**Unterschied zu lvm-manage.sh:**
- `mount-size.sh` = GPT Partitionen (einfacher, direkter)
- `lvm-manage.sh` = LVM Volumes (flexibler, resizebar)

---

## 🚀 Installation

### 1. Script herunterladen/erstellen
```bash
# Script erstellen
nano mount-size.sh
# (Inhalt einfügen)

# Oder von Server kopieren
```

### 2. Ausführbar machen
```bash
chmod +x mount-size.sh
```

### 3. Optional: In System-Pfad verschieben
```bash
sudo cp mount-size.sh /usr/local/bin/mount-size
sudo chmod +x /usr/local/bin/mount-size
```

---

## 📖 Schnelleinstieg (3 Minuten)

### Schritt 1: Disk-Info überprüfen
```bash
# Zeige alle Disks
lsblk

# Output:
# sdc           8:32    0   12T  0 disk
# ├─sdc1        8:33    0 1500G  0 part /part/temp
# └─sdc2        8:34    0  500G  0 part /part/temp2
```

### Schritt 2: Erste Partition erstellen (1000GB)
```bash
sudo ./mount-size.sh -m /part/data -d /dev/sdc -s 1000G
```

### Schritt 3: Zweite Partition erstellen (500GB)
```bash
sudo ./mount-size.sh -m /part/backup -d /dev/sdc -s 500G
```

### Schritt 4: Status überprüfen
```bash
lsblk /dev/sdc
df -h /part/data /part/backup
```

---

## 🎯 Alle Befehle

### Partitionen erstellen

#### Basis Partition erstellen
```bash
sudo ./mount-size.sh -m /part/data -d /dev/sdc -s 1000G
```

**Parameter:**
- `-m` Mountpoint (z.B. /part/data)
- `-d` Device (z.B. /dev/sdc, /dev/sdd)
- `-s` Größe (100M, 500G, 1T, etc.)

**Was passiert:**
- ✅ Prüft ob Disk vorhanden ist
- ✅ Initialisiert mit GPT (wenn nötig)
- ✅ Erstellt neue Partition mit exakter Größe
- ✅ Formatiert mit ext4
- ✅ Erstellt Mountpoint
- ✅ Mountet automatisch
- ✅ Fügt zu /etc/fstab ein

**Wartet auf Bestätigung:**
```
Möchtest du fortfahren? (ja/nein): ja
```

---

#### Mit Custom Dateisystem
```bash
sudo ./mount-size.sh -m /part/data -d /dev/sdc -s 1000G -t xfs
```

**Parameter zusätzlich:**
- `-t` Dateisystem (ext4, xfs, btrfs, ext3)

**Beispiele:**
```bash
# ext4 (Standard, empfohlen)
./mount-size.sh -m /part/data -d /dev/sdc -s 1000G

# XFS (für große Dateien)
./mount-size.sh -m /part/data -d /dev/sdc -s 1000G -t xfs

# btrfs (experimentell)
./mount-size.sh -m /part/data -d /dev/sdc -s 1000G -t btrfs
```

---

### Partitionen löschen

#### Disk komplett wipe (löscht ALLE Partitionen)
```bash
sudo ./mount-size.sh --wipe-disk -d /dev/sdc
```

**Was passiert:**
- ⚠️ Unmountet alle Partitionen
- ⚠️ Löscht ALLE Partitionen auf der Disk
- ✅ Initialisiert GPT neu
- ✅ Disk ist bereit für neue Partitionen

**Fragt zur Bestätigung:**
```
Möchtest du fortfahren? (ja/nein): ja
```

---

#### Disk komplett überschreiben (Sicherheit)
```bash
sudo ./mount-size.sh --wipe-all -d /dev/sdc
```

**⚠️ WARNUNG:** Das dauert STUNDEN bei großen Disks!

**Was passiert:**
- ⚠️ Überschreibt die KOMPLETTE Disk mit Nullen
- ⚠️ Dauert bei 12TB: 2-4 Stunden
- ⚠️ Daten sind absolut nicht wiederherstellbar
- ✅ Initialisiert GPT neu
- ✅ Disk ist "sauber"

**Fragt mehrfach zur Bestätigung:**
```
Wirklich die gesamte Disk überschreiben? (ja/nein): ja
Bist du WIRKLICH sicher? Wiederhole 'JA': JA
```

---

## 💼 Praxisbeispiele

### Szenario 1: Neue 12TB Disk partitionieren (3 Kunden)
```bash
# 1. Erste Partition 1500GB für Kunde A
sudo ./mount-size.sh -m /part/kunde-a -d /dev/sdc -s 1500G

# 2. Zweite Partition 1000GB für Kunde B
sudo ./mount-size.sh -m /part/kunde-b -d /dev/sdc -s 1000G

# 3. Dritte Partition 500GB für Kunde C
sudo ./mount-size.sh -m /part/kunde-c -d /dev/sdc -s 500G

# 4. Überprüfen
lsblk /dev/sdc
df -h /part/kunde-*

# Result:
# /dev/sdc1  1500G  (sdc1 = Kunde A)
# /dev/sdc2  1000G  (sdc2 = Kunde B)
# /dev/sdc3   500G  (sdc3 = Kunde C)
# Freier Platz: 8TB für zukünftige Kunden
```

---

### Szenario 2: Alte Disk ersetzen
```bash
# 1. Neue Disk komplett löschen
sudo ./mount-size.sh --wipe-disk -d /dev/sdd

# 2. Neue Partitionen erstellen
sudo ./mount-size.sh -m /part/storage1 -d /dev/sdd -s 2000G
sudo ./mount-size.sh -m /part/storage2 -d /dev/sdd -s 3000G
sudo ./mount-size.sh -m /part/storage3 -d /dev/sdd -s 2000G

# 3. Fertig!
```

---

### Szenario 3: Verschiedene Dateisysteme
```bash
# ext4 für allgemeine Nutzung
sudo ./mount-size.sh -m /part/standard -d /dev/sdc -s 1000G -t ext4

# XFS für große Backups
sudo ./mount-size.sh -m /part/backups -d /dev/sdc -s 2000G -t xfs

# btrfs für Snapshots (experimentell)
sudo ./mount-size.sh -m /part/snapshots -d /dev/sdc -s 1500G -t btrfs
```

---

### Szenario 4: Mehrere Disks nutzen
```bash
# Disk 1 (/dev/sdc) - 12TB
sudo ./mount-size.sh -m /part/sdc-part1 -d /dev/sdc -s 3000G
sudo ./mount-size.sh -m /part/sdc-part2 -d /dev/sdc -s 3000G
sudo ./mount-size.sh -m /part/sdc-part3 -d /dev/sdc -s 3000G

# Disk 2 (/dev/sdd) - 16TB
sudo ./mount-size.sh -m /part/sdd-part1 -d /dev/sdd -s 5000G
sudo ./mount-size.sh -m /part/sdd-part2 -d /dev/sdd -s 5000G

# Überblick
lsblk | grep -E "sdc|sdd"
```

---

## 📊 Größen-Formate

Das Script versteht verschiedene Größen-Angaben:

```bash
# Megabyte
./mount-size.sh -m /part/test -d /dev/sdc -s 500M

# Gigabyte (empfohlen)
./mount-size.sh -m /part/test -d /dev/sdc -s 1000G

# Terabyte
./mount-size.sh -m /part/test -d /dev/sdc -s 2T
```

**Konvertierung:**
- 1TB = 1000G = 1,000,000M
- 500GB = 500G
- 100GB = 100G

---

## 🔍 Wichtige Infos zu GPT

### GPT vs MBR
```bash
# Dieses Script nutzt GPT (besser für große Disks!)

# MBR Limit: max 2TB pro Partition
# GPT Limit: praktisch unbegrenzt (Exabyte-Bereich)

# Überprüfe welches Format deine Disk hat:
sudo fdisk -l /dev/sdc | grep "Disklabel type"
```

### Maximale Partitionen
```bash
# Mit GPT kannst du beliebig viele Partitionen erstellen
# Praktisch: max 10-20 Partitionen pro Disk sind sinnvoll
# Pro Disk: Standard 3-5 Partitionen
```

---

## 📁 Dateien & Struktur

```
/dev/sdc                    # Disk
├─ /dev/sdc1               # Partition 1 (Größe: z.B. 1000G)
├─ /dev/sdc2               # Partition 2 (Größe: z.B. 500G)
└─ /dev/sdc3               # Partition 3 (Größe: z.B. 3000G)

/part/                      # Mountpoints (empfohlener Pfad)
├─ /part/temp              # gemountet auf sdc1
├─ /part/backup            # gemountet auf sdc2
└─ /part/storage           # gemountet auf sdc3

/etc/fstab                  # Automatisch aktualisiert
# UUID=xxx /part/temp ext4 defaults,nofail 0 2
# UUID=yyy /part/backup ext4 defaults,nofail 0 2
```

---

## 🔐 Sicherheit & Best Practices

### Immer Backup machen
```bash
# Bevor du Partitionen erstellst:
# 1. Backup aktuelle Daten (falls vorhanden)
# 2. Test auf Testdisk machen
# 3. Erst dann auf Production-Disk
```

---

### Disk-Info IMMER überprüfen
```bash
# VOR der Partitionserstellung
sudo lsblk
sudo fdisk -l /dev/sdc

# Stelle sicher, dass es die RICHTIGE Disk ist!
# Falsche Disk kann zu Datenverlust führen!
```

---

### Mountpoint-Konvention
```bash
# Empfohlen:
/part/temp          # Temporäre Daten
/part/backup        # Backups
/part/storage       # Lagerspeicher
/part/data          # Hauptdaten

# NICHT empfohlen:
/backup1, /backup2  # Zu generisch
/mnt/disk1, /mnt/disk2  # Verwirrt mit USB-Sticks
```

---

## 🆘 Troubleshooting

### Problem: "Device nicht gefunden"
```bash
# Überprüfe ob Disk existiert
lsblk

# Oder überprüfe spezifisch
ls -la /dev/sdc

# Wenn nicht vorhanden: Disk ist nicht angeschlossen
# oder hat einen anderen Namen (/dev/sdd, /dev/sde, etc.)
```

---

### Problem: "Schreibfehler" beim Partitionieren
```bash
# Wahrscheinlich: Disk ist noch in Benutzung

# Überprüfe Mounts
mount | grep sdc

# Unmounte falls nötig:
sudo umount /dev/sdc1

# Dann nochmal versuchen:
sudo ./mount-size.sh --wipe-disk -d /dev/sdc
```

---

### Problem: Partition wird nicht erkannt
```bash
# Erzwinge Kernel-Update
sudo partprobe /dev/sdc

# Warte 2 Sekunden
sleep 2

# Überprüfe
lsblk /dev/sdc
```

---

### Problem: Mount fehlgeschlagen
```bash
# Überprüfe Mountpoint
sudo ls -la /part/data/

# Überprüfe /etc/fstab
cat /etc/fstab | grep sdc

# Manuelle Überprüfung
sudo mount -a

# Detaillierter Check
sudo mount /dev/sdc1 /part/data/
```

---

### Problem: Falscher Mountpoint erstellt
```bash
# Nachträglich ändern (mit Vorsicht!)

# 1. Unmounte
sudo umount /part/data

# 2. Editiere fstab
sudo nano /etc/fstab
# Ändere Pfad von /part/data zu /part/newpath

# 3. Erstelle neuer Mountpoint
sudo mkdir -p /part/newpath

# 4. Remounte
sudo mount -a
```

---

## 📊 Überwachung

### Speichernutzung überprüfen
```bash
# Alle Partitionen
df -h

# Spezifische Partition
df -h /part/temp

# Output:
# Filesystem                Size Used Avail Use% Mounted on
# /dev/sdc1                1000G  2.1G  998G   1% /part/temp
```

---

### Partitionsinfo anzeigen
```bash
# Übersicht aller Partitionen
lsblk

# Detaillierte Info
sudo fdisk -l /dev/sdc

# Spezifische Partition
sudo fdisk -l /dev/sdc1
```

---

### Automatische Überwachung (Cron)
```bash
# Als root
sudo crontab -e

# Hinzufügen:
0 2 * * * df -h | grep "/part" >> /var/log/partition-check.log

# Log anschauen
tail -f /var/log/partition-check.log
```

---

## 🔄 Vergleich: mount-size.sh vs lvm-manage.sh

| Feature | mount-size.sh | lvm-manage.sh |
|---------|---------------|---------------|
| **Komplexität** | Einfach | Mittel |
| **Partitionsanzahl** | Begrenzt (~10-20) | Unbegrenzt |
| **Resizable** | Nein ❌ | Ja ✅ |
| **Größenflexibilität** | Mittel | Hoch |
| **Setup-Zeit** | Schnell | Länger |
| **Best für** | Einfache Setups | Flexible Kundenverwaltung |
| **Production** | Ja | Empfohlen |

---

### Wann welches Script?

**Nutze mount-size.sh für:**
- Schnelle, einfache Partitionierung
- Datenverwaltung auf großen Disks
- Temporäre Speicher-Strukturen
- Test & Entwicklung

**Nutze lvm-manage.sh für:**
- Kundenspezifische Speicher-Quotas
- Häufige Größen-Änderungen
- Enterprise-Backup-Systeme
- Flexible Speicherverwaltung

---

## 💡 Tipps & Tricks

### Schnelle Übersicht erstellen
```bash
# Alias für schnelle Überprüfung
alias check-parts='lsblk && echo "---" && df -h /part'

# Nutzen:
check-parts
```

---

### Batch-Partitionierung
```bash
# Script zum Erstellen von mehreren Partitionen
cat > create-partitions.sh << 'EOF'
#!/bin/bash
sudo ./mount-size.sh -m /part/data1 -d /dev/sdc -s 2000G
sudo ./mount-size.sh -m /part/data2 -d /dev/sdc -s 3000G
sudo ./mount-size.sh -m /part/data3 -d /dev/sdc -s 2000G
EOF

chmod +x create-partitions.sh
./create-partitions.sh
```

---

### Wiederverwendbare Vorlage
```bash
# Speichern als template.sh
#!/bin/bash
# Anpassen:
DEVICE=$1         # z.B. /dev/sdc
MOUNTPOINT=$2     # z.B. /part/data
SIZE=$3           # z.B. 1000G

sudo /root/mount-size.sh -m $MOUNTPOINT -d $DEVICE -s $SIZE
```

---

## 📞 Häufige Fragen (FAQ)

### F: Kann ich eine Partition später vergrößern?
**A:** Mit mount-size.sh **NEIN**. Nutze `lvm-manage.sh` für Resizing.

---

### F: Was passiert wenn die Disk voll ist?
**A:** Neue Partitionen können nicht erstellt werden. Nutze eine neue Disk oder lösche alte Partitionen.

---

### F: Kann ich ext4 zu XFS konvertieren?
**A:** Nein, würde Datenverlust verursachen. Du musst Daten sichern, neu formatieren, Daten zurück.

---

### F: Wie viele Partitionen empfohlen?
**A:** 3-5 Partitionen pro Disk sind optimal. Mehr wird schwer zu verwalten.

---

### F: Funktioniert das auch mit NVMe Disks?
**A:** Ja! NVMe funktioniert gleich wie SATA. Nutze `/dev/nvme0n1p1` statt `/dev/sdc1`.

---

## 📚 Zusätzliche Ressourcen

### Linux Befehle die nützlich sind
```bash
# Disk-Info
lsblk              # Übersicht
fdisk -l           # Detailliert
blkid              # UUIDs

# Mounten
mount              # Anzeigen
mount -a           # Alle mounten
umount             # Unmounten

# Dateisystem
df -h              # Speicher
du -sh /path       # Verbrauch
fsck               # Überprüfung
```

---

## 📝 Changelog

### Version 1.0
- ✅ Partitionserstellung mit GPT
- ✅ Automatisches Formatieren
- ✅ Mount & fstab Integration
- ✅ Disk wipe Funktionen
- ✅ Mehrere Dateisysteme
- ✅ Sicherheitsabfragen

---

## 🎯 Best Practices Checkliste

- [ ] Script ist ausführbar: `chmod +x mount-size.sh`
- [ ] Immer als root/sudo nutzen
- [ ] Disk-Name VOR Ausführung überprüfen
- [ ] Backup machen bevor große Operationen
- [ ] fstab Einträge regelmäßig überprüfen
- [ ] Speichernutzung monitoren
- [ ] Größen-Planung im Voraus machen
- [ ] Mountpoint-Konvention einhalten

---

## 🚀 Next Steps

1. **Script installieren** und testen
2. **Eine Test-Disk** partitionieren
3. **Dokumentation lesen** (dieses README)
4. **Production-Setup** planen
5. **Monitoring einrichten** (optional)

---

**Viel Erfolg mit deiner Partitionsverwaltung!** 🚀
