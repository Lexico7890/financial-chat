# 🔧 Diagnóstico de Recursos

Antes de procesar grandes cantidades de PDFs, es recomendable verificar los recursos disponibles en tu sistema para optimizar el rendimiento.

## 🏃‍♂️ Verificación Rápida

### Opción 1: Desde Docker (Recomendado)
```bash
# Diagnóstico completo con recomendaciones
docker compose run ingestion python scripts/diagnose_resources.py

# Con estimación para número específico de PDFs
docker compose run ingestion python scripts/diagnose_resources.py 10000
```

### Opción 2: Directamente en el Host
```bash
# Si tienes Python instalado localmente
python scripts/diagnose_resources.py
```

### Opción 3: Verificación Manual
```bash
# Recursos del contenedor
docker compose run ingestion bash -c "
  echo 'CPUs:' \$(nproc)
  echo 'Memoria:' \$(free -h | grep '^Mem:' | awk '{print \$2\" total, \"\$7\" disponible\"}')
  echo 'Disco:' \$(df -h / | tail -1 | awk '{print \$4\" libre\"}')
"

# Monitoreo en tiempo real
docker stats --no-stream
```

## 📊 Interpretación de Resultados

El script te dará recomendaciones como:

```python
# Ejemplo de configuración optimizada
all_docs = process_pdfs_in_parallel(
    pdf_keys,
    max_workers=6,    # Basado en CPUs disponibles
    batch_size=50     # Basado en memoria disponible
)
```

## ⚙️ Configuración Manual

Si necesitas ajustar manualmente, usa estas guías:

### max_workers (Hilos concurrentes)
- **2-4 CPUs**: `max_workers=2-3`
- **4-8 CPUs**: `max_workers=4-6`
- **8+ CPUs**: `max_workers=6-10`

### batch_size (PDFs por lote)
- **< 4GB RAM**: `batch_size=10-20`
- **4-8GB RAM**: `batch_size=30-50`
- **8-16GB RAM**: `batch_size=50-100`
- **16+ GB RAM**: `batch_size=100-200`

## 🚨 Solución de Problemas

### Error de Memoria (OOMKilled)
```bash
# Reducir configuración
max_workers=2
batch_size=10
```

### Procesamiento Lento
```bash
# Verificar recursos disponibles
docker stats
# Aumentar workers si hay CPUs libres
max_workers=6
```

### Error de Espacio en Disco
```bash
# Limpiar contenedores no utilizados
docker system prune -f
# Verificar espacio
df -h
```

## 📈 Monitoreo Durante el Procesamiento

```bash
# Terminal 1: Ejecutar procesamiento
docker compose run ingestion python ingestion/process_s3_documents.py

# Terminal 2: Monitorear recursos
watch -n 1 'docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"'
```

```python
# Configuración conservadora
max_workers=2
batch_size=20
```

---

💡 **Consejo**: Ejecuta siempre el diagnóstico en el entorno donde planeas hacer el procesamiento, ya que los recursos pueden variar entre desarrollo y producción.