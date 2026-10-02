# argocd-deployanillos

Demo de **despliegue por anillos** con ArgoCD + k3s: la misma app en dos clientes, cada uno en la versión que tú decidas desde Git.

## Estructura

```
base/                      # deployment + service + cloudflared (común a todos)
release/
  v1/                      # base + index.html de v1 (azul, "Versión estable")
  v2/                      # base + index.html de v2 (verde, "Nuevo: módulo de reportes")
rings/
  cliente-a/               # Anillo 1 · Early adopter  -> apunta a una release
  cliente-b/               # Anillo 2 · General        -> apunta a una release
argocd/
  hola-anillos-appset.yaml # ApplicationSet: una Application por carpeta en rings/
```

## Registrar (una sola vez, en el servidor)

```bash
kubectl apply -f https://raw.githubusercontent.com/vicoscanseco/argocd-deployanillos/main/argocd/hola-anillos-appset.yaml
kubectl -n argocd get applications
```

## Promover una versión

1. **Anillo 1:** en `rings/cliente-a/kustomization.yaml` cambia `../../release/v1` → `../../release/v2`, commit + push.
2. Valida Cliente A.
3. **Anillo 2:** mismo cambio en `rings/cliente-b/kustomization.yaml`, commit + push.

Rollback de un cliente = regresar su línea a la release anterior.
Cliente nuevo = copiar una carpeta en `rings/` y ajustar `cliente.json`.

## Ver cada cliente

```bash
for c in cliente-a cliente-b; do echo -n "$c: "; kubectl logs -n $c deploy/cloudflared | grep -o 'https://[a-z0-9-]*\.trycloudflare\.com' | tail -1; done
```

---
VC.DevAI · DevSecOps · Azure · IA · Automatizaciones
