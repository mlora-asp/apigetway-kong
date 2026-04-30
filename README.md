# asp-pago-kong

Proyecto de infraestructura local para exponer Kong como gateway delante de `asp-pago-integration`.

## Responsabilidad

- validar el JWT emitido por un microservicio externo de autenticacion
- aplicar control de trafico
- limpiar headers sensibles
- inyectar headers internos confiables hacia el Bridge
- enrutar solicitudes hacia `asp-pago-integration`

## Levantar solo Kong

```bash
docker-compose up --build
```

Por defecto este compose asume que el Bridge ya esta disponible en `http://host.docker.internal:8080`.

## Endpoints locales

- Proxy: `http://localhost:8000`
- Admin API: `http://localhost:8001`

## Archivo declarativo

La configuracion vive en `kong.yml`.

## Contrato esperado del token

En el flujo actual el cliente llama a Kong con:

```http
Authorization: Bearer <jwt>
```

Para desarrollo local, `kong.yml` está preparado con una credencial JWT `HS256`
para el consumer `asp-pagos-auth-service`.

Claims mínimos esperados:

```json
{
  "iss": "asp-pagos-auth-service",
  "sub": "user-123",
  "scope": "pagos:consulta"
}
```

Kong valida la firma del token, aplica `rate-limiting`, inyecta el secreto
interno `X-Gateway-Secret` y reenvía la petición al Bridge.
