# Trabalho-de-Programa-o
Sistema bancário conforme as especificações do professor, feito por Vitor Dezem e Lucas Basse Pozzer
TAXA_SAQUE = 2.50
TAXA_DEPOSITO = 1.00
TAXA_TRANSFER_PERC = 0.01
TAXA_TRANSFER_MIN = 2.00

ADMIN_SENHA = "vl2025"
ARQ_DADOS = "bancovl_listas.txt"

contas = []
proximo_numero = 1001

def encontrar_indice_conta_por_numero(numero):
    i = 0
    while i < len(contas):
        if contas[i][0] == numero:
            return i
        i = i + 1
    return -1

def ler_int(mensagem):
    valor_str = input(mensagem).strip()
    if valor_str.isdigit() or (valor_str.startswith("-") and valor_str[1:].isdigit()):
        return int(valor_str)
    print("Valor inválido. Digite um número inteiro.")
    return None

def ler_float(mensagem):
    valor_str = input(mensagem).strip()
    ponto = valor_str.replace(".", "", 1)
    if ponto.isdigit() or (valor_str.startswith("-") and ponto[1:].isdigit()):
        return float(valor_str)
    print("Valor inválido. Digite um número (use ponto para decimais).")
    return None

def formatar_dinheiro(valor):
    return "R$ " + str(round(valor, 2))

def registrar_transacao(conta, descricao):
    conta[4].append(descricao)

def salvar_dados():
    linhas = []
    i = 0
    while i < len(contas):
        c = contas[i]
        transacoes = ""
        if len(c[4]) > 0:
            transacoes = ";;".join(c[4])
        linha = str(c[0]) + "|" + c[1].replace("|","/") + "|" + c[2].replace("|","/") + "|" + str(c[3]) + "|" + transacoes
        linhas.append(linha)
        i = i + 1
    arq = open(ARQ_DADOS,"w",encoding="utf-8")
    j = 0
    while j < len(linhas):
        arq.write(linhas[j])
        arq.write(chr(10))
        j = j + 1
    arq.close()
    print("Dados salvos em " + ARQ_DADOS)

def carregar_dados():
    global contas, proximo_numero
    contas = []
    maior_num = 1000
    arq = open(ARQ_DADOS,"r",encoding="utf-8")
    texto = arq.read()
    arq.close()
    linhas = texto.split(chr(10))
    k = 0
    while k < len(linhas):
        linha = linhas[k].strip()
        if linha != "":
            partes = linha.split("|")
            if len(partes) >= 5:
                numero = int(partes[0])
                nome = partes[1]
                senha = partes[2]
                saldo = float(partes[3])
                trans_str = partes[4]
                transacoes = []
                if trans_str != "":
                    transacoes = trans_str.split(";;")
                contas.append([numero,nome,senha,saldo,transacoes])
                if numero > maior_num:
                    maior_num = numero
        k = k + 1
    proximo_numero = maior_num + 1
    print("Dados carregados de " + ARQ_DADOS)

def cadastrar_conta():
    global proximo_numero
    print("=== Cadastro de Conta ===")
    nome = input("Nome completo: ").strip()
    if nome == "":
        print("Nome inválido.")
        return
    senha = input("Defina uma senha (mín. 4 caracteres): ").strip()
    if len(senha) < 4:
        print("Senha muito curta.")
        return
    saldo_str = input("Saldo inicial (ENTER para 0): ").strip()
    saldo = 0.0
    if saldo_str != "":
        ponto = saldo_str.replace(".","",1)
        if ponto.isdigit():
            saldo = float(saldo_str)
            if saldo < 0:
                print("Saldo inicial não pode ser negativo.")
                return
        else:
            print("Valor inválido para saldo inicial.")
            return
    numero = proximo_numero
    proximo_numero = proximo_numero + 1
    conta = [numero,nome,senha,saldo,[]]
    registrar_transacao(conta,"Conta criada para " + nome + " com saldo " + formatar_dinheiro(saldo))
    contas.append(conta)
    print("Conta criada! Número: " + str(numero))

def autenticar():
    print("=== Login ===")
    numero = ler_int("Número da conta: ")
    if numero is None:
        return None
    senha = input("Senha: ").strip()
    idx = encontrar_indice_conta_por_numero(numero)
    if idx == -1:
        print("Conta não encontrada.")
        return None
    c = contas[idx]
    if c[2] == senha:
        print("Bem-vindo(a), " + c[1])
        return c
    print("Credenciais inválidas.")
    return None

def consultar_saldo(conta):
    print("Saldo atual: " + formatar_dinheiro(conta[3]))

def mostrar_extrato(conta):
    print("=== Extrato ===")
    if len(conta[4]) == 0:
        print("Nenhuma movimentação.")
        return
    i = 0
    while i < len(conta[4]):
        print(conta[4][i])
        i = i + 1

def depositar(conta):
    print("=== Depósito ===")
    valor = ler_float("Valor do depósito: ")
    if valor is None or valor <= 0:
        print("Valor deve ser positivo.")
        return
    taxa = TAXA_DEPOSITO
    conta[3] = conta[3] + valor - taxa
    registrar_transacao(conta,"Depósito de " + formatar_dinheiro(valor) + " | Taxa: " + formatar_dinheiro(taxa) + " | Novo saldo: " + formatar_dinheiro(conta[3]))
    print("Depósito realizado. Taxa aplicada: " + formatar_dinheiro(taxa))
    consultar_saldo(conta)

def sacar(conta):
    print("=== Saque ===")
    valor = ler_float("Valor do saque: ")
    if valor is None or valor <= 0:
        print("Valor deve ser positivo.")
        return
    total = valor + TAXA_SAQUE
    if conta[3] < total:
        print("Saldo insuficiente. Necessário " + formatar_dinheiro(total) + ".")
        return
    conta[3] = conta[3] - total
    registrar_transacao(conta,"Saque de " + formatar_dinheiro(valor) + " | Taxa: " + formatar_dinheiro(TAXA_SAQUE) + " | Novo saldo: " + formatar_dinheiro(conta[3]))
    print("Saque realizado com sucesso.")
    consultar_saldo(conta)

def transferir(conta_origem):
    print("=== Transferência ===")
    numero_destino = ler_int("Número da conta destino: ")
    if numero_destino is None:
        return
    idx_dest = encontrar_indice_conta_por_numero(numero_destino)
    if idx_dest == -1 or contas[idx_dest][0] == conta_origem[0]:
        print("Conta destino inválida.")
        return
    valor = ler_float("Valor da transferência: ")
    if valor is None or valor <= 0:
        print("Valor deve ser positivo.")
        return
    taxa = valor * TAXA_TRANSFER_PERC
    if taxa < TAXA_TRANSFER_MIN:
        taxa = TAXA_TRANSFER_MIN
    total = valor + taxa
    if conta_origem[3] < total:
        print("Saldo insuficiente. Necessário " + formatar_dinheiro(total) + " (inclui taxa de " + formatar_dinheiro(taxa) + ").")
        return
    conta_origem[3] = conta_origem[3] - total
    contas[idx_dest][3] = contas[idx_dest][3] + valor
    registrar_transacao(conta_origem,"Transferência para " + str(contas[idx_dest][0]) + " de " + formatar_dinheiro(valor) + " | Taxa: " + formatar_dinheiro(taxa) + " | Novo saldo: " + formatar_dinheiro(conta_origem[3]))
    registrar_transacao(contas[idx_dest],"Recebimento de transferência de " + str(conta_origem[0]) + " de " + formatar_dinheiro(valor) + " | Novo saldo: " + formatar_dinheiro(contas[idx_dest][3]))
    print("Transferência realizada com sucesso.")
    consultar_saldo(conta_origem)

def listar_saldos_auditoria():
    print("=== Listagem de Saldos (Acesso somente por VL) ===")
    senha = input("Senha de administrador: ").strip()
    if senha != ADMIN_SENHA:
        print("Senha incorreta.")
        return
    if len(contas) == 0:
        print("Nenhuma conta cadastrada.")
        return
    print("Número  | Nome                        | Saldo (R$)")
    print("--------------------------------------------------------")
    i = 0
    while i < len(contas):
        c = contas[i]
        num_str = str(c[0]).ljust(8)
        nome_str = c[1][:28].ljust(28)
        saldo_str = str(round(c[3],2)).rjust(12)
        print(num_str + " | " + nome_str + " | " + saldo_str)
        i = i + 1

def menu_logado(conta):
    sair = False
    while not sair:
        print("=== Menu da Conta " + str(conta[0]) + " ===")
        print("1) Consultar saldo")
        print("2) Depositar")
        print("3) Sacar")
        print("4) Transferir")
        print("5) Extrato")
        print("0) Sair da conta")
        opcao = input("Escolha: ").strip()
        if opcao == "1":
            consultar_saldo(conta)
        elif opcao == "2":
            depositar(conta)
        elif opcao == "3":
            sacar(conta)
        elif opcao == "4":
            transferir(conta)
        elif opcao == "5":
            mostrar_extrato(conta)
        elif opcao == "0":
            print("Saindo da conta...")
            sair = True
        else:
            print("Opção inválida.")

def menu_principal():
    sair = False
    while not sair:
        print("====================================")
        print("          SISTEMA BANCO VL          ")
        print("====================================")
        print("1) Cadastrar conta")
        print("2) Login")
        print("3) Listar saldos (Acesso somete por VL)")
        print("4) Salvar dados")
        print("5) Carregar dados")
        print("0) Sair")
        opcao = input("Escolha: ").strip()
        if opcao == "1":
            cadastrar_conta()
        elif opcao == "2":
            c = autenticar()
            if c is not None:
                menu_logado(c)
        elif opcao == "3":
            listar_saldos_auditoria()
        elif opcao == "4":
            salvar_dados()
        elif opcao == "5":
            carregar_dados()
        elif opcao == "0":
            print("Encerrando o sistema. Até mais!")
            sair = True
        else:
            print("Opção inválida.")

if __name__ == "__main__":
    menu_principal()
