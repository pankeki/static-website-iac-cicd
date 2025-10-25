# Statiskas vietnes izvietošana uz AWS (Terraform + GitHub Actions)

Šis repo satur gatavu piemēru, kā izvietot statisku tīmekļa vietni AWS izmantojot:
- **Terraform** (infrastruktūra kā kods)
- **AWS S3** (objektu glabāšana)
- **AWS CloudFront** (CDN)
- **GitHub Actions** (CI/CD: izvietošanas plūsma)

> **Piezīme:** pārbaudi un seko instrukcijām zemāk, pirms izvieto dzīvajā AWS kontā. Terraform var radīt maksas resursus.

---

## Struktūra
```
/terraform         # Terraform konfigurācija (S3, CloudFront)
index.html         # Statiskā mājaslapa
styles.css         # Vienkārša stilistika
.github/workflows/  # GitHub Actions workflows (infra + deploy)
```

---

## Kā palaist lokāli (soļi)

1. **Instalē rīkus**  
   - Terraform (>=1.2)  
   - AWS CLI  
   - Git

2. **Uzstādi AWS CLI piekļuvi lokālai darbībai**  
   ```
   aws configure
   ```
   Ievadi `AWS Access Key ID`, `AWS Secret Access Key`, `region` (piem., `eu-central-1`).

3. **Izveido infrastruktūru ar Terraform**  
   ```
   cd terraform
   terraform init
   terraform apply
   ```
   Pēc apstiprināšanas Terraform izveidos S3 spaini un CloudFront izplatīšanu.

4. **Pārsūti statisko saturu uz S3**  
   No repo saknes:
   ```
   aws s3 sync . s3://<BUCKET_NAME> --exclude ".git/*" --delete
   ```
   Tu vari iegūt `<BUCKET_NAME>` no Terraform izvades (`terraform output s3_bucket_id`) vai no `terraform/outputs.tf`.

5. **Pārbaudi CloudFront domēnu**  
   Terraform outputs izvadīs CloudFront domēnu — atver to pārlūkprogrammā (`https://<CLOUDFRONT_DOMAIN>`).

---

## CI/CD (GitHub Actions)

Repo satur `.github/workflows/deploy.yml` — kad tu izveidos GitHub repozitoriju un iestatīsi GitHub Secrets, katrs push uz `main`:
- sinhronizēs repozitorija saturu uz S3, un
- izveidos CloudFront invalidāciju.

### Nepieciešamie GitHub Secrets
- `AWS_ACCESS_KEY_ID`  
- `AWS_SECRET_ACCESS_KEY`  
- `AWS_REGION` (piem., `eu-central-1`)  
- `S3_BUCKET` (S3 spainis, ko izveidoji ar Terraform)  
- `CLOUDFRONT_DISTRIBUTION_ID` (no Terraform output)

---

## Drošība un optimizācija
- Pēc testa vari iestatīt S3 objektu dzīves ciklu (lētākai glabāšanai).
- Ja vēlēsies pielāgot domēnu un SSL certifikātu, pievieno Route53 un ACM konfigurāciju Terraform.

---
## Link
- https://d208htktay4o0u.cloudfront.net/