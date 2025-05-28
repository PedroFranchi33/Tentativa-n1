import json
import os
import hashlib
import webbrowser
import re
from statistics import mean, median, mode
import matplotlib.pyplot as plt
import statistics  

ARQUIVO_DADOS = "usuarios.json"

def limpar_tela():
    os.system('cls' if os.name == 'nt' else 'clear')

def inicializar_arquivo_dados():
    if not os.path.exists(ARQUIVO_DADOS):
        with open(ARQUIVO_DADOS, 'w') as arquivo:
            json.dump({"usuarios": {}, "perguntas": []}, arquivo, indent=4)

def gerar_hash_senha(senha):
    return hashlib.sha256(senha.encode()).hexdigest()

def validar_senha(senha):
    return (
        len(senha) >= 8 and
        any(c.isupper() for c in senha) and
        any(not c.isalnum() for c in senha)
    )
    

def carregar_dados():
    inicializar_arquivo_dados()
    try:
        with open(ARQUIVO_DADOS, 'r') as arquivo:
            dados = json.load(arquivo)
    except json.JSONDecodeError:
        dados = {"usuarios": {}, "perguntas": []}
        salvar_dados(dados)

    if "usuarios" not in dados:
        dados["usuarios"] = {}
    if "perguntas" not in dados:
        dados["perguntas"] = []

    salvar_dados(dados)
    return dados

def salvar_dados(dados):
    with open(ARQUIVO_DADOS, 'w') as arquivo:
        json.dump(dados, arquivo, indent=4)

def registrar_usuario():
    limpar_tela()
    nome_usuario = input("Digite o nome de usuário: ")
    
    while True:
        senha = input("Digite a senha (mínimo de 8 caracteres, uma letra maiúscula e um caractere especial): ")
        
        if validar_senha(senha):
            break
        else:
            print("Erro: A senha deve ter pelo menos 8 caracteres, uma letra maiúscula e um caractere especial!")

    dados = carregar_dados()
    if nome_usuario in dados["usuarios"]:
        print("Erro: Nome de usuário já existe!")
        return

    dados["usuarios"][nome_usuario] = {
        "senha": gerar_hash_senha(senha),
        "contagem_acessos": 0,
        "pontuacoes": []
    }
    salvar_dados(dados)
    print("Usuário registrado com sucesso!")

def login_usuario():
    limpar_tela()
    nome_usuario = input("Digite o nome de usuário: ")
    senha = input("Digite a senha: ")

    dados = carregar_dados()
    if nome_usuario in dados["usuarios"] and dados["usuarios"][nome_usuario]["senha"] == gerar_hash_senha(senha):
        dados["usuarios"][nome_usuario]["contagem_acessos"] += 1
        salvar_dados(dados)
        print("Login bem-sucedido!")
        return nome_usuario
    else:
        print("Erro: Nome de usuário ou senha inválidos!")
        return None

def registrar_pergunta():
    limpar_tela()
    pergunta = input("Digite a pergunta: ")
    resposta = input("Digite a resposta correta: ")

    dados = carregar_dados()
    dados["perguntas"].append({"pergunta": pergunta, "resposta": resposta})
    salvar_dados(dados)
    print("Pergunta registrada com sucesso!")

def listar_usuarios():
    limpar_tela()
    dados = carregar_dados()
    if not dados["usuarios"]:
        print("Nenhum usuário cadastrado.")
        return

    print("\nUsuários cadastrados:")
    for nome in dados["usuarios"]:
        print(f"- {nome}")

def realizar_quiz(nome_usuario):
    quizzes = {
        "Tecnologia da Informação": [
            {
                "pergunta": "O que significa 'CPU'?",
                "alternativas": ["1) Central Process Unit", "2) Central Processing Unit", "3) Computer Personal Unit", "4) Central Performance Unit"],
                "resposta": "2"
            },
            {
                "pergunta": "Qual desses é um sistema operacional?",
                "alternativas": ["1) Python", "2) Linux", "3) HTML", "4) SQL"],
                "resposta": "2"
            },
            {
                "pergunta": "O que é memória RAM?",
                "alternativas": ["1) Armazenamento permanente", "2) Memória de acesso aleatório", "3) Memória só de leitura", "4) Disco rígido"],
                "resposta": "2"
            },
            {
                "pergunta": "O que faz um firewall?",
                "alternativas": ["1) Acelera a internet", "2) Bloqueia acessos não autorizados", "3) Salva arquivos", "4) Atualiza o sistema"],
                "resposta": "2"
            },
            {
                "pergunta": "Qual a função do HD?",
                "alternativas": ["1) Processar dados", "2) Armazenar dados", "3) Gerar energia", "4) Controlar rede"],
                "resposta": "2"
            }
        ],
        "Programação Básica e Boas Práticas": [
            {
                "pergunta": "Qual linguagem é usada para estilizar páginas web?",
                "alternativas": ["1) HTML", "2) CSS", "3) JavaScript", "4) Python"],
                "resposta": "2"
            },
            {
                "pergunta": "O que é um 'loop' em programação?",
                "alternativas": ["1) Condição", "2) Repetição", "3) Variável", "4) Função"],
                "resposta": "2"
            },
            {
                "pergunta": "Qual dessas é uma boa prática de programação?",
                "alternativas": ["1) Escrever código confuso", "2) Usar nomes claros para variáveis", "3) Evitar comentários", "4) Repetir código desnecessariamente"],
                "resposta": "2"
            },
            {
                "pergunta": "Para que serve um 'debugger'?",
                "alternativas": ["1) Escrever código", "2) Encontrar erros no código", "3) Compilar o programa", "4) Excluir arquivos"],
                "resposta": "2"
            },
            {
                "pergunta": "Qual símbolo é usado para comentário em Python?",
                "alternativas": ["1) //", "2) <!-- -->", "3) #", "4) /* */"],
                "resposta": "3"
            }
        ],
        "Segurança Digital e Cibersegurança": [
            {
                "pergunta": "O que é phishing?",
                "alternativas": ["1) Tipo de malware", "2) Tentativa de fraude online", "3) Firewall", "4) Ataque físico"],
                "resposta": "2"
            },
            {
                "pergunta": "Qual dessas senhas é mais segura?",
                "alternativas": ["1) senha123", "2) 123456", "3) Qw!8zP#2", "4) abcdef"],
                "resposta": "3"
            },
            {
                "pergunta": "O que significa VPN?",
                "alternativas": ["1) Virtual Private Network", "2) Very Private Network", "3) Virtual Public Network", "4) Verified Private Network"],
                "resposta": "1"
            },
            {
                "pergunta": "O que é um firewall?",
                "alternativas": ["1) Programa que bloqueia acessos não autorizados", "2) Tipo de vírus", "3) Rede Wi-Fi pública", "4) Dispositivo de armazenamento"],
                "resposta": "1"
            },
            {
                "pergunta": "Qual é uma boa prática para evitar invasões?",
                "alternativas": ["1) Usar mesma senha em vários sites", "2) Atualizar sistemas regularmente", "3) Ignorar avisos de segurança", "4) Compartilhar senhas"],
                "resposta": "2"
            }
        ]
    }

    materias = list(quizzes.keys())

    while True:
        print("\n--- Matérias disponíveis para o quiz ---")
        for i, materia in enumerate(materias, 1):
            print(f"{i}. {materia}")
        print(f"{len(materias)+1}. Voltar ao menu")

        escolha = input("Escolha uma matéria para fazer o quiz: ")

        if escolha.isdigit():
            escolha = int(escolha)
            if escolha == len(materias) + 1:
                break
            elif 1 <= escolha <= len(materias):
                materia_escolhida = materias[escolha - 1]
                perguntas = quizzes[materia_escolhida]

                pontuacao = 0
                for i, p in enumerate(perguntas):
                    print(f"\nPergunta {i+1}: {p['pergunta']}")
                    for alt in p["alternativas"]:
                        print(alt)
                    resposta = input("Digite o número da alternativa correta: ").strip()
                    if resposta == p["resposta"]:
                        pontuacao += 1
                
                dados = carregar_dados()
                dados["usuarios"][nome_usuario]["pontuacoes"].append(pontuacao)
                salvar_dados(dados)

                print(f"\nVocê acertou {pontuacao} de {len(perguntas)} perguntas da matéria '{materia_escolhida}'.")
                input("Pressione Enter para continuar...")
                limpar_tela()
            else:
                print("Opção inválida. Tente novamente.")
        else:
            print("Opção inválida. Tente novamente.")

def gerar_estatisticas(nome_usuario):
    limpar_tela()
    dados = carregar_dados()
    
    usuario = dados["usuarios"].get(nome_usuario)
    if not usuario or not usuario["pontuacoes"]:
        print("Nenhuma pontuação disponível para gerar estatísticas.")
        input("\nPressione Enter para continuar...")
        return
        
    pontuacoes =  usuario ["pontuacoes"]

    print(f"Estatísticas de desempenho de '{nome_usuario}':\n")
    print(f"Média das pontuações: {mean(pontuacoes):.2f}")
    print(f"Mediana das pontuações: {median(pontuacoes):.2f}")

    input("\nPressione Enter para continuar...")

def listar_perguntas():
    limpar_tela()
    dados = carregar_dados()
    if not dados["perguntas"]:
        print("Nenhum quiz cadastrado.")
        return
    print("\nQuizzes cadastrados:")
    for i, p in enumerate(dados["perguntas"]):
        print(f"{i+1}. Pergunta: {p['pergunta']}, Resposta: {p['resposta']}")

def editar_pergunta():
    dados = carregar_dados()
    listar_perguntas()
    try:
        indice = int(input("Digite o número do quiz para editar: ")) - 1
        if 0 <= indice < len(dados["perguntas"]):
            nova_pergunta = input("Digite a nova pergunta: ")
            nova_resposta = input("Digite a nova resposta correta: ")
            dados["perguntas"][indice]["pergunta"] = nova_pergunta
            dados["perguntas"][indice]["resposta"] = nova_resposta
            salvar_dados(dados)
            print("Quiz editado com sucesso!")
        else:
            print("Índice inválido!")
    except ValueError:
        print("Entrada inválida. Digite um número.")

def excluir_pergunta():
    dados = carregar_dados()
    listar_perguntas()
    try:
        indice = int(input("Digite o número do quiz para excluir: ")) - 1
        if 0 <= indice < len(dados["perguntas"]):
            del dados["perguntas"][indice]
            salvar_dados(dados)
            print("Quiz excluído com sucesso!")
        else:
            print("Índice inválido!")
    except ValueError:
        print("Entrada inválida. Digite um número.")

def excluir_usuario():
    dados = carregar_dados()
    listar_usuarios()
    nome_para_excluir = input("Digite o nome do usuário a ser excluído: ")
    if nome_para_excluir in dados["usuarios"]:
        del dados["usuarios"][nome_para_excluir]
        salvar_dados(dados)
        print(f"Usuário '{nome_para_excluir}' excluído com sucesso!")
    else:
        print(f"Erro: Usuário '{nome_para_excluir}' não encontrado!")

def acessar_materias():
    materias = {
        "1": {
            "nome": "Tecnologia da Informação",
            "material": "https://drive.google.com/file/d/1YJ6DBnrueWhg32lvXNvFvpd-AgDz7XGZ/view?usp=drive_link",
            "video": "https://youtu.be/WULwZ6v_Ai8?si=NBjn5BPs4FL0jCGc"
        },
        "2": {
            "nome": "Programação Básica e Boas Práticas",
            "material": "https://drive.google.com/file/d/1XMFxEKHsO2OPiwpxAeRqJ8FLb60K2KQS/view?usp=drive_link",
            "video": "https://youtu.be/4p7axLXXBGU?si=b0em9zHW2b9eLXpn"
        },
        "3": {
            "nome": "Segurança Digital e Cibersegurança",
            "material": "https://drive.google.com/file/d/1Kb4ec1ukgg9ni5qHOKJMb-QciVMUX0eq/view?usp=drive_link",
            "video": "https://youtu.be/Gfh2bxe3hGU?si=7Q3nwqkzVxm6bnuP"
        }
    }

    while True:
        limpar_tela()
        print("\n--- Matérias Disponíveis ---")
        for chave, info in materias.items():
            print(f"{chave}. {info['nome']}")
        print("4. Voltar ao menu anterior")

        escolha = input("Escolha uma matéria para continuar: ")

        if escolha in materias:
            materia = materias[escolha]
            while True:
                limpar_tela()
                print(f"\n--- {materia['nome'].upper()} ---")
                print("1. Acessar material teórico")
                print("2. Acessar vídeo aula")
                print("3. Voltar à lista de matérias")
                sub_opcao = input("Escolha uma opção: ")

                if sub_opcao == "1":
                    print(f"Abrindo material: {materia['material']}")
                    webbrowser.open(materia["material"])
                elif sub_opcao == "2":
                    print(f"Abrindo vídeo aula: {materia['video']}")
                    webbrowser.open(materia["video"])
                elif sub_opcao == "3":
                    break
                else:
                    print("Opção inválida. Tente novamente.")
        elif escolha == "4":
            break
        else:
            print("Opção inválida. Tente novamente.")

def gerar_grafico_estatistico(nome_usuario):
    dados = carregar_dados()
    usuario = dados["usuarios"].get(nome_usuario)

    if not usuario or not usuario.get("pontuacoes"):
        print("Nenhuma pontuação disponível para gerar o gráfico.")
        input("\nPressione enter para continuar...")

    pontuacoes = usuario["pontuacoes"]
    medias = [
        ("Média", mean(pontuacoes)),
        ("Mediana", median(pontuacoes))
    ]

    try:
        medias.append(("moda", mode(pontuacoes)))
    except statistics.StatisticsError:
        medias.append(("Moda", 0))

    nome, valores = zip(*medias)

    plt.bar(nome, valores, color=["skyblue", "orange", "lightgreen"])
    plt.title(f"Estatística do usuário {nome_usuario}")
    plt.ylabel("Valor")
    plt.tight_layout()
    plt.savefig("grafico_estatisticas.png")  # Salva como imagem
    plt.show()


def main():
    inicializar_arquivo_dados()

    while True:
        limpar_tela()
        print("\nPlataforma de Educação Digital")
        print("1. Registrar usuário")
        print("2. Fazer login")
        print("3. Sair")
        escolha = input("Digite sua escolha: ")

        if escolha == "1":
            registrar_usuario()
        elif escolha == "2":
            nome_usuario = login_usuario()
            if nome_usuario:
                if nome_usuario == "admin":
                    while True:
                        limpar_tela()
                        print("\nMenu do Administrador")
                        print("1. Registrar pergunta")
                        print("2. Listar usuários cadastrados")
                        print("3. Listar quizzes")
                        print("4. Editar quiz")
                        print("5. Excluir quiz")
                        print("6. Excluir usuário")
                        print("7. Sair")
                        escolha_admin = input("Digite sua escolha: ")
                        if escolha_admin == "1":
                            registrar_pergunta()
                        elif escolha_admin == "2":
                            listar_usuarios()
                        elif escolha_admin == "3":
                            listar_perguntas()
                        elif escolha_admin == "4":
                            editar_pergunta()
                        elif escolha_admin == "5":
                            excluir_pergunta()
                        elif escolha_admin == "6":
                            excluir_usuario()
                        elif escolha_admin == "7":
                            break
                        else:
                            print("Opção inválida! Tente novamente.")
                else:
                    while True:
                        limpar_tela()
                        print("\nMenu do Usuário")
                        print("1. Acessar Matérias")
                        print("2. Acessar Quizzes")
                        print("3. Gerar estatísticas")
                        print("4. Gerar gráfico estatistico")
                        print("5. Sair")
                        escolha_usuario = input("Digite sua escolha: ")

                        if escolha_usuario == "1":
                            acessar_materias()
                        elif escolha_usuario == "2":
                            realizar_quiz(nome_usuario)
                        elif escolha_usuario == "3":
                            gerar_estatisticas(nome_usuario)
                        elif escolha_usuario == "4":
                            gerar_grafico_estatistico(nome_usuario)
                        elif escolha_usuario == "5":
                            break
                        else:
                            print("Opção inválida! Tente novamente.")
                            input("\nPressione Enter para continuar.")
        elif escolha == "3":
            print("Saindo do programa...")
            break
        else:
            print("Opção inválida! Tente novamente.")

if __name__ == "__main__":
    main()
