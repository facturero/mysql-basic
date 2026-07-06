# mysql-basic

Proyecto de **infraestructura básica** (no es un microservicio) para desplegar y versionar **MySQL 9** en tu cluster **k3s**, usando GitOps simple: cualquier cambio en `k8s/` que se suba a `main` se aplica automáticamente vía el runner self-hosted de GitHub Actions.

## ¿Qué hace?

- Despliega **MySQL 9** (`mysql-basic`) con almacenamiento persistente (`PersistentVolumeClaim`, storage class `local-path` de k3s).
- Lo expone vía `NodePort` para conectarte desde fuera del cluster (DBeaver, `mysql` client, tu backend, etc).

## Puerto expuesto

| Servicio | Puerto interno | NodePort | Acceso |
|---|---|---|---|
| MySQL | 3306 | **30306** | `mysql -h <ip-servidor> -P 30306 -u <user> -p` |

## Configuración inicial (una sola vez)

1. Crea el repo en GitHub y conéctalo:

   ```bash
   cd C:\Users\sansh\cmr-proyect\backend\mysql-basic
   git init
   git branch -M main
   git remote add origin <URL_DE_TU_REPO>
   ```

2. Agrega estos **Secrets** en GitHub (Settings → Secrets and variables → Actions):

   | Secret | Ejemplo |
   |---|---|
   | `MYSQL_ROOT_PASSWORD` | contraseña root fuerte |
   | `MYSQL_DATABASE` | `crm_dev` |
   | `MYSQL_USER` | `crm_user` |
   | `MYSQL_PASSWORD` | contraseña de la app |

3. Confirma que tu runner self-hosted tiene `kubectl` configurado apuntando a tu cluster k3s (normalmente vía `KUBECONFIG=~/.kube/config`, como quedó en la instalación).

4. Sube el proyecto:

   ```bash
   git add .
   git commit -m "infra: mysql-basic"
   git push -u origin main
   ```

## Flujo día a día

```bash
git add k8s/
git commit -m "infra: ajuste de recursos mysql-basic"
git push
```

El runner aplica los cambios automáticamente. También puedes disparar el deploy manualmente desde la pestaña **Actions** de GitHub (`workflow_dispatch`).

## Verificar estado manualmente

```bash
kubectl get pods -l app=mysql-basic
kubectl logs deployment/mysql-basic
```

## Notas

- **Recursos:** límites de memoria/CPU ajustados pensando en un servidor con ~7GB de RAM (ver `resources` en `k8s/mysql-basic-deployment.yaml`).
- **Backups:** el PVC sobrevive a caídas de pod y reinicios, pero NO es un backup real. Considera un cronjob de `mysqldump` hacia S3.
- **Namespace:** se despliega en `default`.
