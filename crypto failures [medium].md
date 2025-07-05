# 🧠 TryHackMe - CryptoFailures Writeup

## 📌 Información general

* **Sala**: CryptoFailures (suponiendo nombre basado en la URL)
* **Enfoque**: Criptografía rota / Autenticación rota / Manipulación de cookies
* **Nivel**: Intermedio
* **Objetivo**: Obtener acceso como `admin` sin conocer su clave secreta, explotando la debilidad en la autenticación basada en cookies

---

## 🔍 Análisis inicial

Tras acceder a la URL principal, observamos que el sistema genera dos cookies:

* `user=guest`
* `secure_cookie=<hash_largo>`

El archivo `index.php.bak` revela la lógica de validación:

```php
$string = $user . ":" . $_SERVER['HTTP_USER_AGENT'] . ":" . $ENC_SECRET_KEY;
$salt = substr($_COOKIE['secure_cookie'], 0, 2);
if (make_secure_cookie($string, $salt) === $_COOKIE['secure_cookie']) {
    if ($user === "admin") {
        echo "congrats: FLAG";
    }
}
```

---

## 🧪 Vulnerabilidad detectada

* El `secure_cookie` se genera usando `crypt()` por bloques de 8 caracteres, usando un `salt` visible (los dos primeros caracteres del hash)
* El campo `user` se extrae directamente de la cookie sin validación adicional
* El sistema **confía ciegamente** en la combinación `user` + `secure_cookie` sin verificar que sean coherentes entre sí

---

## 🔓 Ataque 1: Fuerza bruta de `ENC_SECRET_KEY` para guest

Se desarrolló un script en Python para generar `secure_cookie` en base a cadenas del tipo:

```
guest:<User-Agent>:<clave>
```

Con ello, se descubrió que la clave usada para `guest` era:

```
ENC_SECRET_KEY = 123456
```

---

## 🔓 Ataque 2: Generación de cookie completa para `admin` (fallido)

Se intentó generar una cookie completa para `admin` usando la misma clave `123456`, pero el sistema respondió con:

```
You are not logged in
```

➡️ Lo que indica que `admin` usa **una clave secreta distinta**.

---

## 💥 Ataque real: Bypass parcial por manipulación del primer bloque

Gracias a un writeup externo, se probó la siguiente técnica:

### 🎯 Técnica:

1. Tomar una cookie real generada para `guest`
2. Sustituir solo el **primer bloque** del hash (correspondiente a `guest:Mo`)
3. Reemplazarlo por el bloque correspondiente a `admin:Mo` calculado con el mismo `salt`
4. Establecer `user=admin`

```php
$salt = substr($cookie,0,2);
$guest_part = crypt("guest:Mo", $salt);
$admin_part = crypt("admin:Mo", $salt);
$modified_cookie = str_replace($guest_part, $admin_part, $cookie);
```

### ✅ Resultado:

Con la cookie modificada, el sistema aceptó el acceso y respondió con:

```
congrats: THM{...FLAG...}
```

➡️ El servidor **no validaba la coherencia entre `user` y el resto del hash**, lo que permitió un bypass de autenticación.

---

## 📚 Lecciones aprendidas

* Nunca confíes en campos enviados por el cliente como `user` sin firmarlos (HMAC, JWT, etc.)
* Los hashes sin integridad (como `crypt()` con `DES`) pueden ser manipulados por bloques
* La seguridad lógica es tan importante como la criptográfica

---

## ✅ Estado final

* `ENC_SECRET_KEY` de guest: `123456`
* Bypass por manipulación parcial: exitoso
* Flag obtenida: **✅**

---

## ✍️ Autor del writeup

* Usuario: *h4cksito*
* Fecha: 2025-07-05
* Certificación objetivo: eJPTv2

---

¿Te gustó esta técnica? ¡Sígueme en Twitter: [@h4cksito](https://twitter.com/h4cksito) para más writeups como este! 🐚
