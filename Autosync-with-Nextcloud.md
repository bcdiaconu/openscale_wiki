## Requirements

* [Nextcloud App](https://play.google.com/store/apps/details?id=com.nextcloud.client)
* [Nextcloud Server](https://nextcloud.com/) (reachable and with an user account)

## Settings in openScale

1. Settings - Backup
2. Export folder: Check if there is a separate folder for your openScale backup
3. Schedule: weekly
4. Automatic backup: activated

The backup will now be saved with the creation-date in the specified folder.

## Settings in Nextcloud
1. Connect to your Nextcloud server
2. (optional) Create a new folder for backups
3. "Automatic uploads"
4. Create a new folder (on top)
5. Local folder: choose your specified openScale backup folder
6. Remote folder: choose your folder in the Nextcloud instance where your backups should be located
7. Save

With that, openScale will backup automatically and the Nextcloud App will upload it to your Nextcloud profile.