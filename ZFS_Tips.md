### List volumes
```
sudo zfs list
```
### List snapshots
```
sudo zfs list -t snapshot
```
### Create snapshot
```
sudo zfs snapshot volme@snapshotname
```
### Delete snapshot
```
sudo zfs destroy volume@snapshotname
```
### Restore a snapshot
```
sudo zfs rollback volume@snapshotname
```
