# p2p
codigo
import socket
import threading

# Endereço IP e porta para comunicação entre os nós
HOST = '192.168.100.25'
PORTA = 12345

# Função para determinar o vencedor
def determinar_vencedor(escolha_cliente, escolha_servidor):
    resultados = {
        ('pedra', 'pedra'): 'Empate',
        ('pedra', 'papel'): 'Você perdeu',
        ('pedra', 'tesoura'): 'Você ganhou',
        ('papel', 'pedra'): 'Você ganhou',
        ('papel', 'papel'): 'Empate',
        ('papel', 'tesoura'): 'Você perdeu',
        ('tesoura', 'pedra'): 'Você perdeu',
        ('tesoura', 'papel'): 'Você ganhou',
        ('tesoura', 'tesoura'): 'Empate'
    }

    return resultados[(escolha_cliente, escolha_servidor)]

# Função para lidar com as mensagens recebidas de outros nós
def lidar_com_mensagens(conn, addr, modo):
    escolha_servidor = None

    while True:
        try:
            # Aguarda a escolha do usuário
            if escolha_servidor is not None:
                # Ambos jogaram, exibe as escolhas dos jogadores
                if modo == 's':
                    print(f"\nVocê escolheu: {escolha_servidor}")
                    print(f"O oponente escolheu: {escolha_cliente}\n")
                else:
                    print(f"\nVocê escolheu: {escolha_cliente}")
                    print(f"O oponente escolheu: {escolha_servidor}\n")

                # Reinicia as escolhas para o próximo jogo
                escolha_servidor = None
            else:
                escolha_servidor = input("Digite sua escolha (pedra, papel, tesoura): ").strip().lower()
                conn.sendall(escolha_servidor.encode())

                # Recebe a escolha do outro jogador
                escolha_cliente = conn.recv(1024).decode().strip().lower()
                if not escolha_cliente:
                    print("Conexão encerrada pelo servidor.")
                    break

                # Envia a escolha do cliente de volta
                conn.sendall(escolha_cliente.encode())

        except Exception as e:
            print(f"Erro: {e}")
            break

# Função principal do servidor
def servidor():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        # Liga o socket ao endereço e porta especificados
        s.bind((HOST, PORTA))
        # Começa a ouvir por conexões recebidas
        s.listen()
        print(f"Servidor ouvindo em {HOST}:{PORTA}")

        # Aceita a conexão do cliente
        conn, addr = s.accept()
        print(f"Cliente conectado por {addr}")

        # Inicia uma thread para lidar com as mensagens recebidas do cliente
        threading.Thread(target=lidar_com_mensagens, args=(conn, addr, 's')).start()

# Função principal do cliente
def cliente():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        # Conecta-se ao servidor
        s.connect((HOST, PORTA))
        print(f"Conectado ao servidor {HOST}:{PORTA}")

        # Inicia uma thread para lidar com as mensagens recebidas do servidor
        threading.Thread(target=lidar_com_mensagens, args=(s, None, 'c')).start()

        # Loop principal do cliente
        while True:
            try:
                # Recebe a escolha do outro jogador
                escolha_servidor = s.recv(1024).decode().strip().lower()
                if not escolha_servidor:
                    print("Conexão encerrada pelo servidor.")
                    break

                # Aguarda a escolha do usuário
                escolha_cliente = input("Digite sua escolha (pedra, papel, tesoura): ").strip().lower()
                s.sendall(escolha_cliente.encode())

                # Recebe a escolha do servidor
                escolha_servidor = s.recv(1024).decode().strip().lower()
                if not escolha_servidor:
                    print("Conexão encerrada pelo servidor.")
                    break

            except Exception as e:
                print(f"Erro: {e}")
                break

# Função principal
def main():
    # Determina se o script será servidor ou cliente
    modo = input("Você deseja ser servidor (s) ou cliente (c)? ").strip().lower()

    if modo == 's':
        servidor()
    elif modo == 'c':
        cliente()
    else:
        print("Opção inválida. Escolha 's' para servidor ou 'c' para cliente.")

if __name__ == '__main__':
    main()
