# mysql-basic

Proyecto de **infraestructura básica** (no es un microservicio) para desplegar y versionar **MySQL 9** en tu cluster **k3s**, usando GitOps simple: cualquier cambio en `k8s/` que se suba a `master` se aplica automáticamente vía el runner self-hosted de GitHub Actions.

## ¿Qué hace?

- Despliega **MySQL 9** (`mysql-basic`) con almacenamiento persistente (`PersistentVolumeClaim`, storage class `local-path` de k3s).
- Lo expone vía `NodePort` para conectarte desde fuera del cluster (DBeaver, `mysql` client, tu backend, etc).

## Puerto expuesto

| Servicio | Puerto interno | NodePort | Acceso |
|---|---|---|---|
| MySQL | 3306 | **30306** | `mysql -h <ip-servidor> -P 30306 -u <user> -p` |

## Configuración inicial (una sola vez)

1. Agrega estos **Secrets** en GitHub (Settings → Secrets and variables → Actions) del repo `mysql-basic`:

   | Secret | Ejemplo |
   |---|---|
   | `MYSQL_ROOT_PASSWORD` | contraseña root fuerte |
   | `MYSQL_DATABASE` | `crm_dev` |
   | `MYSQL_USER` | `crm_user` |
   | `MYSQL_PASSWORD` | contraseña de la app |

2. Confirma que tu runner self-hosted tiene `kubectl` configurado apuntando a tu cluster k3s (normalmente vía `KUBECONFIG=~/.kube/config`, como quedó en la instalación).

3. Sube el proyecto:

   ```bash
   git add .
   git commit -m "infra: mysql-basic"
   git push -u origin master
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

## Si el auto-despliegue no se dispara

- Confirma en GitHub → pestaña **Actions** si aparece el workflow "Deploy mysql-basic to k3s" corriendo o con error.
- Confirma que tu runner self-hosted está **online** (Settings → Actions → Runners, debe verse verde/"Idle").
- Confirma que el push fue a la rama `master` (la que espera el trigger) y que tocó archivos dentro de `k8s/`.
- Si el runner está online pero el job falla en el paso de `kubectl`, revisa que el usuario que corre el runner tenga acceso al `kubeconfig` de k3s (variable `KUBECONFIG` visible para el servicio, no solo para tu sesión interactiva).

## Notas

- **Recursos:** límites de memoria/CPU ajustados pensando en un servidor con ~7GB de RAM (ver `resources` en `k8s/mysql-basic-deployment.yaml`).
- **Backups:** el PVC sobrevive a caídas de pod y reinicios, pero NO es un backup real. Considera un cronjob de `mysqldump` hacia S3.
- **Namespace:** se despliega en `default`.
