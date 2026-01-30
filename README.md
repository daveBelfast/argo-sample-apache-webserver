# Apache Webserver Kubernetes Deployment

Dieses Projekt enthält Kubernetes Manifeste zum Deployment eines Apache Webservers mit LoadBalancer Service.

## Übersicht

Das Projekt besteht aus zwei separaten Manifest-Dateien:
- **deployment.yaml**: Deployment-Konfiguration für den Apache Webserver
- **service.yaml**: Service-Konfiguration (LoadBalancer) für den externen Zugriff

## Komponenten

### Deployment
- **Name**: apache-webserver
- **Replicas**: 3
- **Image**: httpd:2.4
- **Container Port**: 80
- **Ressourcen**:
  - Requests: 100m CPU, 128Mi Memory
  - Limits: 500m CPU, 512Mi Memory
- **Health Checks**:
  - Liveness Probe: HTTP GET auf Port 80
  - Readiness Probe: HTTP GET auf Port 80

### Service
- **Name**: apache-webserver-service
- **Type**: LoadBalancer
- **Port**: 10000 (extern)
- **Target Port**: 80 (Container)
- **NodePort**: 30080

## Installation

### Voraussetzungen
- Kubernetes Cluster (z.B. Minikube, GKE, EKS, AKS)
- kubectl installiert und konfiguriert
- Zugriff auf einen Cluster mit LoadBalancer-Unterstützung

### Deployment durchführen

1. **Deployment erstellen**:
   ```bash
   kubectl apply -f deployment.yaml
   ```

2. **Service erstellen**:
   ```bash
   kubectl apply -f service.yaml
   ```

   Oder beide Dateien gleichzeitig:
   ```bash
   kubectl apply -f .
   ```

## Überprüfung

### Status des Deployments prüfen
```bash
kubectl get deployments
kubectl get pods -l app=apache
```

### Service und externe IP abrufen
```bash
kubectl get service apache-webserver-service
```

Die externe IP wird unter der Spalte `EXTERNAL-IP` angezeigt. Bei Cloud-Providern kann es einige Minuten dauern, bis die IP zugewiesen wird.

### Logs der Pods anzeigen
```bash
kubectl logs -l app=apache
```

## Zugriff auf den Webserver

Nach erfolgreichem Deployment ist der Apache Webserver über folgende Methoden erreichbar:

1. **LoadBalancer IP** (Cloud-Provider):
   ```bash
   curl http://<EXTERNAL-IP>:10000
   ```

2. **NodePort** (Lokal/Minikube):
   ```bash
   curl http://<NODE-IP>:30080
   ```

## Skalierung

Um die Anzahl der Replicas zu ändern:
```bash
kubectl scale deployment apache-webserver --replicas=5
```

## Deinstallation

Um alle Ressourcen zu entfernen:
```bash
kubectl delete -f .
```

Oder einzeln:
```bash
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
```

## Troubleshooting

### Pods starten nicht
```bash
kubectl describe pod -l app=apache
```

### Service hat keine externe IP
- Bei Minikube: `minikube tunnel` ausführen
- Bei Cloud-Providern: Einige Minuten warten, bis die IP zugewiesen wird

### Webserver nicht erreichbar
1. Firewall-Regeln überprüfen
2. Service-Konfiguration überprüfen: `kubectl describe service apache-webserver-service`
3. Pod-Logs prüfen: `kubectl logs -l app=apache`

## Anpassungen

### HTML-Inhalt hinzufügen
Um eigene HTML-Dateien bereitzustellen, kann ein ConfigMap oder ein PersistentVolume verwendet werden:

```yaml
# In deployment.yaml unter spec.template.spec.containers[0] hinzufügen:
volumeMounts:
  - name: html-content
    mountPath: /usr/local/apache2/htdocs
volumes:
  - name: html-content
    configMap:
      name: apache-html
```

## Lizenz

Dieses Projekt dient zu Demonstrationszwecken.
