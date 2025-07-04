# 🧠 TryHackMe - Cipher | Smart Contract Exploitation Writeup

**Categoría**: Blockchain / Smart Contracts  
**Dificultad**: Intermedia  
**Objetivo**: Drenar el balance de ETH de un contrato vulnerable para que `isSolved()` devuelva `true`.

---

## 🔍 1. Introducción

En este reto, se nos presenta un contrato inteligente Ethereum desplegado en una red privada, accesible mediante un endpoint JSON-RPC (`8545`) y una API auxiliar (`/challenge`) que nos proporciona datos útiles como:

- Clave privada del atacante
- Dirección del contrato
- Dirección del jugador

El contrato simula una tesorería de la botnet "Phantom Node", y nuestro objetivo es sabotearla drenando los fondos del contrato.

---

## 🔎 2. Análisis del contrato

El código fuente es accesible directamente desde la interfaz web (`index.js`) y muestra el siguiente contrato:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract Challenge {
    address private owner;
    address private initOwner;

    constructor() payable {
        owner = msg.sender;
        initOwner = msg.sender;
    }

    function changeOwnership() external {
        owner = msg.sender;
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner!");
        payable(owner).transfer(address(this).balance);
    }

    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }

    function getOwnerBalance() external view returns (uint256) {
        return address(initOwner).balance;
    }

    function isSolved() external view returns (bool) {
        return (address(this).balance == 0);
    }

    function getAddress() external view returns (address) {
        return msg.sender;
    }

    function getOwner() external view returns (address) {
        return owner;
    }
}

⚠️ 3. Vulnerabilidad detectada

La función changeOwnership() no está restringida, lo que permite que cualquier dirección la invoque y se convierta en dueña del contrato:
function changeOwnership() external {
    owner = msg.sender;
}

Este error lógico permite una secuencia de ataque muy simple:

    Tomar el control del contrato.

    Invocar withdraw() para vaciar el balance.

    Confirmar que isSolved() devuelve true.

🚀 4. Explotación con web3.py

Se desarrolló un script en Python para automatizar el ataque, utilizando la clave privada y el ABI mínimo:
# 1. changeOwnership()
# 2. withdraw()
# 3. isSolved()

  ✅ 5. Resultado

Tras ejecutar el ataque, el contrato quedó vacío y la función isSolved() devolvió true, cumpliendo el objetivo del reto.
✅ ¿Challenge resuelto?: True

  🛡️ 6. Mitigación

Para evitar este tipo de vulnerabilidad, la función changeOwnership() debe estar protegida por una restricción de autorización. Por ejemplo:
modifier onlyOwner {
    require(msg.sender == owner, "Not authorized");
    _;
}

function changeOwnership(address newOwner) external onlyOwner {
    owner = newOwner;
}

🧩 7. Conclusión

Este reto refleja un error común y crítico en contratos inteligentes: no controlar quién puede modificar propiedades sensibles. A pesar de su simplicidad, este tipo de fallo ha causado millones en pérdidas reales en proyectos DeFi.
