# Testrepo for Lakefs initialization including MinIO S3 Storage

## 0. Start Lakefs, MinIO and Postgres on Fileserver
docker compose up

## 1. Install and setup lakectl, lakectl local and minIO Client

### MinIO
- MinIO Client installieren
    ```
    curl -O https://dl.min.io/client/mc/release/linux-amd64/mc
    chmod +x mc
    sudo mv mc /usr/local/bin/
    ```
- `mc alias set localminio http://127.0.0.1:9000 minioadmin minioadmin`
- `mc alias set lakefs http://127.0.0.1:8000 AKIAIOSFOLKFSSAMPLES wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`
- Überprüfen: `mc alias list`

### LakeCtl
- LakeCtl Client installieren
    ```
    wget https://github.com/treeverse/lakeFS/releases/download/v1.49.1/lakeFS_1.49.1_Linux_x86_64.tar.gz
    tar xfvz lakeFS_1.49.1_Linux_x86_64.tar.gz
	chmod +x lake*
    sudo cp lakefs /usr/local/bin/
    sudo cp lakectl /usr/local/bin/
    ```
- Konfiguration von Lakectl `lakectl config`
    - Acces Key: `AKIAIOSFOLKFSSAMPLES`
    - Secure Key: `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`
    - URL: `http://127.0.0.1:8000/api/v1`
- Testen: `lakectl repo list`

## 2. Setup MinIO Bucket, LakeFS Repo and upload data:
1. MinIO Bucket erstellen (`mc mb localminio/irisbucket`)
1. LakeFS Repository erstellen (`lakectl repo create lakefs://irisrepo s3://irisbucket`)
1. Create venv and run `python3 get_data.py`
1. Daten uploaden: `mc cp ./data/ lakefs/irisrepo/main/data --recursive`
1. Commit current minio status: `lakectl commit lakefs://irisrepo/main -m "Commit current data version" `

## 2.1 Setup local machine for remote server
1. Set mc aliases:
- `mc alias set remoteminio http://192.168.1.97:9000 minioadmin minioadmin` # Note: This needs to be your remote-server IP
- `mc alias set remotelakefs http://192.168.1.97:8000 AKIAIOSFOLKFSSAMPLES wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` # Note: This needs to be your remote-server IP
2. Set lakectl config
`lakectl config`
    - Acces Key: `AKIAIOSFOLKFSSAMPLES`
    - Secure Key: `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`
    - URL: `http://192.168.1.97:8000/api/v1` # Note: This needs to be your remote-server IP
Note:
- For mc mb ... commands you need to use "remoteminio" instead of "localminio" and "remotelakefs" instead of "lakefs"
- for lakectl commands, since this is configured to be the remote, use lakefs://... and s3://...

## 3. Usage (lakefs local siehe: https://docs.lakefs.io/quickstart/work-with-data-locally.html oder https://docs.lakefs.io/howto/local-checkouts.html)
1. Create Branch: `lakectl branch create lakefs://repo-name/dev --source lakefs://repo-name/main`
2. minio in DNS hinzufügen (um Fehler bei der DNS Auflösung zu vermeiden) `sudo nano /etc/hosts` --> Zeile hinzufügen für `127.0.0.0 minio` # Note: for remote server usage use the remote server IP adress here
3. Git Repo erstellen (falls der Ordner nicht sowieso schon teil eines git Projekts ist): `git init`
4. Zielordner für das Kopieren der Daten erstellen `mkdir data`
5. Clone Data to local folder, also adds the data to the .gitignore. Note: The Project must be git tracked! `lakectl local clone lakefs://repo-name/dev/data/ data`
6. (for later update use `lakectl local checkout`)
7. ... process_data.py
8. `lakectl local status data/` shows the changes
9. commit and push to lakefs `lakectl local commit -m "resizing all data" data`
10. merge back to main branch `lakectl merge lakefs://repo-name/dev/ lakefs://repo-name/main`
11. handle git `git add .` + `git commit -m "first change"`+ `git push`

## Further Commands: https://docs.lakefs.io/reference/cli.html
- `lakectl local list` in git repo root shows the directory that is synced with lakeFS
- `lakectl local init lakefs://repo-name/main/test nameofnewfolder/` to link a new local folder to lakefs

## Reproduce a model
- `git checkout $commitId`

