# 🏁 Google Fly Cup Challenge: Recruit — Arcade Genius Edition

🔗 YouTube: [Arcade Genius YouTube Channel](https://www.youtube.com/@ArcadeGenius-z1)

---

## 🚀 Setup: Load the Dataset in Cloud Shell

```bash
for file in `gsutil ls gs://spls/gsp394/tables/*.csv`; do TABLE_NAME=`echo $file | cut -d '/' -f6 | cut -d '.' -f1`; bq load --autodetect --source_format=CSV --replace=true drl.$TABLE_NAME $file; done.
