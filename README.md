# TerminalPhyton
Equipe Composta por: Luis Henrique Barbosa Moreira Meneses, Mayara Camille Nascimento Silva, Anna Clara Maia Lima Cavalcante e Évelin Yasmin Menezes de Souza
//Código Terminal Phyton: 
import socket
from datetime import datetime

SENHA_PADRAO = "admin123"
HOST_PADRAO = "127.0.0.1"
PORTA_PADRAO = 5000


def processar_comando(comando):
    comando = comando.strip()

    if comando.lower() == "data":
        agora = datetime.now().strftime("%d/%m/%Y %H:%M:%S")
        return f"Data e hora do servidor: {agora}"

    elif comando.lower() == "status":
        return "Servidor ativo e funcionando normalmente."

    elif comando.lower().startswith("msg "):
        mensagem = comando[4:].strip()

        if not mensagem:
            return "Erro: escreva uma mensagem após o comando msg."

        print(f"\n[RECADO DO CLIENTE] {mensagem}")
        return "Mensagem recebida pelo servidor com sucesso!"

    elif comando.lower() == "ajuda":
        return (
            "Comandos disponíveis:\n"
            "data   - Mostra data e hora\n"
            "status - Verifica o servidor\n"
            "msg <texto> - Envia um recado\n"
            "ajuda  - Lista os comandos\n"
            "segredo - Mostra uma mensagem especial\n"
            "sair   - Encerra a conexão"
        )

    elif comando.lower() == "segredo":
        return "Parabéns! Você descobriu o comando secreto!"

    else:
        return "Comando inválido. Digite 'ajuda'."


def iniciar_servidor():
    print("=== TERMINAL DE COMANDOS: SERVIDOR ===")

    host = input(
        f"IP de escuta [{HOST_PADRAO}]: "
    ).strip() or HOST_PADRAO

    try:
        porta_texto = input(
            f"Porta de escuta [{PORTA_PADRAO}]: "
        ).strip()

        porta = int(porta_texto or PORTA_PADRAO)

        if not 1 <= porta <= 65535:
            raise ValueError

    except ValueError:
        print("Erro: informe uma porta entre 1 e 65535.")
        return

    servidor = socket.socket(
        socket.AF_INET, socket.SOCK_STREAM
    )

    try:
        servidor.setsockopt(
            socket.SOL_SOCKET, socket.SO_REUSEADDR, 1
        )
        servidor.bind((host, porta))
        servidor.listen(5)

        print(f"\nServidor escutando em {host}:{porta}")
        print("Aguardando conexões...\n")

        while True:
            conexao, endereco = servidor.accept()

            with conexao:
                print(f"Cliente conectado: {endereco}")

                # Autenticação: no máximo três tentativas.
                arquivo = conexao.makefile(
                    "r", encoding="utf-8", newline="\n"
                )

                try:
                    autenticado = False

                    for tentativa in range(3):
                        senha = arquivo.readline()

                        if not senha:
                            break

                        if senha.rstrip("\r\n") == SENHA_PADRAO:
                            conexao.sendall(
                                b"OK: autenticacao realizada!\n"
                            )
                            autenticado = True
                            break
                        else:
                            restantes = 2 - tentativa
                            resposta = (
                                "ERRO: senha incorreta. "
                                f"Tentativas restantes: {restantes}\n"
                            )
                            conexao.sendall(
                                resposta.encode("utf-8")
                            )

                    if not autenticado:
                        print("Autenticação falhou.")
                        continue

                    print("Cliente autenticado.")

                    while True:
                        linha = arquivo.readline()

                        if not linha:
                            break

                        comando = linha.rstrip("\r\n")

                        if comando.lower() == "sair":
                            conexao.sendall(
                                b"Conexao encerrada pelo cliente.\n"
                            )
                            break

                        resposta = processar_comando(comando)
                        conexao.sendall(
                            (resposta + "\n").encode("utf-8")
                        )

                except (ConnectionError, OSError):
                    print("Conexão interrompida.")
                finally:
                    arquivo.close()

                print("Cliente desconectado.\n")

    except OSError as erro:
        print(f"Erro ao iniciar o servidor: {erro}")
    except KeyboardInterrupt:
        print("\nServidor encerrado.")
    finally:
        servidor.close()


if __name__ == "__main__":
    iniciar_servidor()
