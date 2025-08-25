from web3 import Web3
from eth_account import Account
import time

# Conexión a un nodo (por ejemplo, Infura, Alchemy, etc.)
w3 = Web3(Web3.HTTPProvider("https://mainnet.infura.io/v3/TU_API_KEY"))

# Dirección y ABI de la factory del DEX (ej. Uniswap v2 en Ethereum)
factory_address = Web3.to_checksum_address("0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f")
factory_abi = [
    {
        "anonymous": False,
        "inputs": [
            {"indexed": True, "name": "token0", "type": "address"},
            {"indexed": True, "name": "token1", "type": "address"},
            {"indexed": False, "name": "pair", "type": "address"},
            {"indexed": False, "name": "", "type": "uint256"}
        ],
        "name": "PairCreated",
        "type": "event"
    }
]

factory = w3.eth.contract(address=factory_address, abi=factory_abi)

# Configuración de la cuenta y la clave privada
private_key = "TU_CLAVE_PRIVADA"
account = Account.from_key(private_key)

# Función para comprar el token recién creado
def comprar_token(token_address):
    # Esta función debe implementar:
    # 1. Calcular la ruta de swap (token->ETH o similar)
    # 2. Construir la transacción de compra
    # 3. Firmarla y enviarla
    # Aquí solo mostramos un placeholder
    print(f"Comprando token en {token_address}")
    # ... implementación completa ...

# Escucha continua del evento PairCreated
def escuchar_nuevos_tokens():
    event_filter = factory.events.PairCreated.create_filter(fromBlock="latest")
    print("Esperando nuevos pares...")

    while True:
        for evento in event_filter.get_new_entries():
            token0 = evento["args"]["token0"]
            token1 = evento["args"]["token1"]
            pair_address = evento["args"]["pair"]

            # Ejemplo: si token0 es WETH, el otro es el token nuevo
            weth_address = Web3.to_checksum_address("0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2")
            if token0 == weth_address:
                nuevo_token = token1
            elif token1 == weth_address:
                nuevo_token = token0
            else:
                # No involucra WETH, se ignora
                continue

            print(f"Nuevo token detectado: {nuevo_token}, par: {pair_address}")
            comprar_token(nuevo_token)

        time.sleep(2)  # breve espera antes de la siguiente búsqueda

if __name__ == "__main__":
    escuchar_nuevos_tokens()
