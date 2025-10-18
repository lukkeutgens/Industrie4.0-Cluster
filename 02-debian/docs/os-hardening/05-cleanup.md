# Clean Up Unused Packages
Remove packages that were once installed as dependencies but are no longer needed. 
This helps keep the system lean, reduces clutter, and minimizes potential attack surfaces.

```bash
sudo apt autoremove --purge
sudo apt autoclean
```
