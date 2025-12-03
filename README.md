# Block Storage, Encryption & Filesystem Demo

*förstå **hur lagring fungerar i Linux**:
från rå disk → partition → kryptering → filesystem → mount.

Detta är kärnkunskap i systemadministration och DevOps.

---

# 1. Arkitektur & flöde

```
┌──────────────────────────────┐
│   Virtuell Disk (5GB)        │  ← /dev/sdb
└──────────────┬───────────────┘
               │ partitioning
               ▼
┌──────────────────────────────┐
│   Partition (sdb1)           │  ← /dev/sdb1
└──────────────┬───────────────┘
               │ LUKS-encryption
               ▼
┌──────────────────────────────┐
│   LUKS Encrypted Volume      │  ← /dev/mapper/cryptodisk
└──────────────┬───────────────┘
               │ filesystem creation
               ▼
┌──────────────────────────────┐
│   EXT4 Filesystem            │
└──────────────┬───────────────┘
               │ mount
               ▼
┌──────────────────────────────┐
│   Mountpoint (/mnt)          │
└──────────────────────────────┘
```

**Ordningen är avgörande**:
*block device → partition → encryption → filesystem → mount*

---

# 2. Pay attention part


* vad ett block device är
* hur partitioner fungerar
* hur kryptering lager-på-lager funkar (LUKS)
* varför man skapar filesystem *efter* kryptering
* hur mounting fungerar och varför det “tar över” en katalog
* hur Linux strukturerar diskar under `/dev` och `/dev/mapper`


> “Ni ska förstå flödet.”

---

# 3. Steg-för-steg med kommandon och förklaringar

## 3.1 Kolla befintliga diskar

```bash
lsblk
```

Visar block devices, partitioner och mountpoints.

---

## 3.2 Skapa ny virtuell disk

Gjort via VirtualBox (ej ett Linux-kommando).

---

## 3.3 Partitionera disken (sdb)

Starta `fdisk`:

```bash
sudo fdisk /dev/sdb
```

Inuti fdisk:

| Kommando | Betydelse                      |
| -------- | ------------------------------ |
| `g`      | skapa nytt GPT partition table |
| `n`      | skapa ny partition             |
| `p`      | visa partitionstabellen        |
| `w`      | skriv ändringar till disk      |

Resultat:
`/dev/sdb1` skapas.

---

## 3.4 Skapa LUKS-kryptering

```bash
sudo cryptsetup luksFormat /dev/sdb1
```

Viktigt: måste skriva **YES** i stora bokstäver.

### Förklaring:

* LUKS skapar ett krypterat lager *ovanpå* partitionen.
* Data i `/dev/sdb1` är nu permanent krypterad tills den öppnas.

---

## 3.5 Öppna den krypterade volymen

```bash
sudo cryptsetup open /dev/sdb1 cryptodisk
```

Detta skapar en ny device:

```
/dev/mapper/cryptodisk
```

Den är **inte** krypterad i läst form, OS:et dekrypterar automatiskt när du läser/skriv­er.

---

## 3.6 Skapa filesystem (EXT4)

```bash
sudo mkfs.ext4 /dev/mapper/cryptodisk
```

### Varför först nu?

* Om du skapar filsystemet *innan* kryptering → du krypterar inget.
* Filssystemet måste ligga **ovanpå** krypteringslagret för att fungera korrekt.

---

## 3.7 Skapa mountpoint och mounta

```bash
sudo mount /dev/mapper/cryptodisk /mnt
```

Nu är disken åtkomlig under `/mnt`.

Testfil:

```bash
sudo touch /mnt/MySecretEncryptedFile
```

---

## 3.8 Unmount och stäng volymen

Unmount:

```bash
sudo umount /mnt
```

Stäng krypterad volym:

```bash
sudo cryptsetup close cryptodisk
```

När volymen är stängd är **allt fullständigt krypterat**.

---

# 4. Viktiga begrepp 

## 4.1 Block device

En abstraktion av råa diskar i Linux.

Exempel:

* `/dev/sda`
* `/dev/sdb`
* `/dev/mapper/cryptodisk`

Block devices ger **random access**

---

## 4.2 Partition

En uppdelning av en disk i logiska delar.

---

## 4.3 LUKS (Linux Unified Key Setup)

Standard för disk-kryptering i Linux.

Egenskaper:

* Dekryptering sker i realtid
* Data på disken är *alltid* krypterad
* Passphrase behövs varje gång volymen öppnas

---

## 4.4 Filesystem

EXT4 i detta exempel.

Han nämnde:

* journal
* lost+found
* metadata

---

## 4.5 Mount

Att “fästa” ett filesystem vid en katalog.

När du mountar något till `/mnt` tar det över katalogen.

---

# 5. Vanliga misstag som demonstrerades

* Skapa filsystem innan kryptering → fel ordning
* Glömma `umount` innan `cryptsetup close`
* Inte läsa varningar i `cryptsetup luksFormat`
* Tro att `/mnt` innehållet försvinner — det bara byts tillfälligt

---

# Slutsats
Förstå den logiska strukturen:

```
Disk → Partition → Kryptering → Filesystem → Mount
```

Detta är fundamentet för all lagringsadministration i Linux och en central del i DevOps rollen.

---

# Kommandolista

```bash
lsblk

sudo fdisk /dev/sdb
  g
  n
  w

sudo cryptsetup luksFormat /dev/sdb1
sudo cryptsetup open /dev/sdb1 cryptodisk

sudo mkfs.ext4 /dev/mapper/cryptodisk

sudo mount /dev/mapper/cryptodisk /mnt
sudo touch /mnt/file

sudo umount /mnt
sudo cryptsetup close cryptodisk
```

---

> “Jag vill att ni förstår konceptet.”

Detta är en av de viktigaste demolektionerna i hela Linux-delen av kursen. På riktigt.
