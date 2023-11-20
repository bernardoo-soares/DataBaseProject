###################################################
##########    BIBLIOTECAS A USAR    ###############
###################################################
###################################################
import os
import msvcrt
import psycopg2.extras
from tabulate import tabulate
import time
from datetime import datetime
import hashlib

###################################################
##########    FUNÇÕES MENU INICIAL  ###############
###################################################
###################################################

def login():
    
    num_tentativas = 0

    while num_tentativas < 3:
        # Limpa o terminal
        os.system('cls')

        print("\t\t\t---- LOGIN ----\n")

        # Introduzir dados de autenticação
        email = input("\t\t\tDigite o seu email: ")
        password = input("\t\t\tDigite a sua senha: ")

        #Encriptar password
        password = password.encode('utf-8')
        password = hashlib.sha256(password).hexdigest()
        
        # Verificar se o email e senha correspondem a um usuário válido na base de dados
        cur.execute("SELECT * FROM cliente WHERE email = %s AND password = %s", (email, password))
        cliente = cur.fetchone()
        
        #Verificar se o utilizador 'existe'
        if cliente:
            # Se o utilizador foi encontrado na base de dados, exibe uma mensagem de boas-vindas
            nome = cliente['nome_completo']

            print("\n\t\t\tBem-vindo,", nome, "!")
            print("\n\t\t\tPressione qualquer tecla para avançar...")
            msvcrt.getch()

            #Chamar funçao Menu Cliente
            menu_cliente(cliente)
            #Voltar ao Menu Principal
            return
        else:
            num_tentativas += 1
            print("\n\t\t\tEmail ou senha inválidos!")

            if num_tentativas == 3:
                print("\n\t\t\tExcedeu o número de tentativas!")
                print("\n\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
                msvcrt.getch()
                return
        
                
        # Opção para voltar ao menu principal
        voltar_menu = input("\n\t\t\tPressione 'm' para voltar ao Menu Principal ou qualquer outra tecla para introduzir novos dados... ")
        if voltar_menu.lower() == "m":
            return

def registo():
    while True:
        # Limpa o terminal
        os.system('cls')

        print("\t\t\t---- NOVO REGISTO ----\n")

        # Pede as informações do novo cliente
        nome = input("\n\t\t\tInsira o seu nome completo: ")
        nif = input("\t\t\tInsira o seu NIF: ")
        telefone = input("\t\t\tInsira o seu número de telefone: ")
        email = input("\t\t\tInsira o seu endereço de email: ")
        password = input("\t\t\tInsira a sua password desejada: ")
        confirm_password = input("\t\t\tConfirme a sua password: ")

        # Verifica se o NIF ou email já existem na base de dados
        cur.execute("SELECT nif, email FROM cliente WHERE nif = %s OR email = %s", (nif, email))
        resultado_pesquisa = cur.fetchone()

        if resultado_pesquisa is not None:
            existing_nif = resultado_pesquisa['nif']
            existing_email = resultado_pesquisa['email']
 

            if existing_nif == int(nif):
                print("\n\t\t\tJá existe um cliente registado com esse NIF. Tente novamente.")
            elif existing_email == email:
                print("\n\t\t\tJá existe um cliente registado com esse email. Tente novamente.")
            elif password != confirm_password:
                print("\n\t\t\tAs passwords não correspondem. Tente novamente.")

            voltar_menu = input("\n\t\t\tPressione 'm' para voltar ao Menu Principal ou qualquer outra tecla para introduzir novos dados... ")
            
            if voltar_menu.lower() == "m":
                return
            else:
                continue


        #Encriptar a password
        password = password.encode('utf-8')
        password = hashlib.sha256(password).hexdigest()

        # Insere as informações do novo cliente na base de dados
        cur.execute("INSERT INTO cliente (nome_completo, nif, telefone, email, password, tipo_gold) VALUES (%s, %s, %s, %s, %s, %s)", (nome, nif, telefone, email, password, False))
        conn.commit()
        #Nota: Quando um novo cliente se regista, o estatuto Gold não é ativado

        print("\n\n\t\t\tRegisto efetuado com sucesso!")
        print("\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
        msvcrt.getch()
        return

def admin_login():
    num_tentativas = 0

    while num_tentativas < 3:
        # Limpa o terminal
        os.system('cls')

        print("\t\t\t---- LOGIN ADMINISTRADOR----\n")

        # Introduzir dados de autenticação
        email = input("\t\t\tDigite o seu email: ")
        password = input("\t\t\tDigite a sua senha: ")

        #Encriptar password
        password = password.encode('utf-8')
        password = hashlib.sha256(password).hexdigest()
        
        # Verificar se o email e senha correspondem a um usuário válido na base de dados
        cur.execute("SELECT * FROM administrador WHERE email = %s AND password = %s", (email, password))
        admin = cur.fetchone()
        
        #Verificar se o admin 'existe'
        if admin:
            nome = admin['nome_completo']

            print("\n\t\t\tCredenciais administrativas válidas! Bem-vindo, admin", nome, "!")
            print("\n\t\t\tPressione qualquer tecla para avançar...")
            msvcrt.getch()

            #Chamar funçao Menu Admin
            admin_menu(admin)

            #Voltar ao Menu Principal
            return
        else:
            num_tentativas += 1
            print("\n\t\t\tCredenciais administrativas inválidas!")

            if num_tentativas == 3:
                print("\n\t\t\tExcedeu o número de tentativas!")
                print("\n\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
                msvcrt.getch()
                return
        
                
        # Opção para voltar ao menu principal
        voltar_menu = input("\n\t\t\tPressione 'm' para voltar ao Menu Principal ou qualquer outra tecla para reintroduzir dados... ")
        if voltar_menu.lower() == "m":
            return
         


###################################################
##########    MENU ADMINISTRADOR ##################
###################################################
###################################################

def admin_menu(admin):
    while True:
        # Limpa o terminal
        os.system('cls')

        nome = admin['nome_completo']
        print(f"\t\t\tMENU ADMINISTRADOR --- {nome}")
        print("\t\t\t1) Gerir viagens")
        print("\t\t\t2) Gerir frota de autocarros")
        print("\t\t\t3) Enviar notificação")
        print("\t\t\t4) Consultar clientes e gerir estatutos")
        print("\t\t\t5) Ver estatísticas")
        print("\t\t\t6) Logout\n")
        
        option = input("\t\t\tSelecione uma opção: ")

        
        if option == "1":
            # Opção 1 - Gerir viagens
            gerir_viagens()
            
        elif option == "2":
            # Opção 2 - Gerir a frota de autocarros
            gerir_autocarros()
            
        elif option == "3":
            # Opção 3 - Enviar notificação
            mensagem_admin(admin)

        elif option == "4":
            # Opção 4 - Consultar clientes e gerir estatutos
            gerir_clientes()
            
        elif option == "5":
            # Opção 5 - Ver estatísticas
            ver_estatisticas()

        elif option == "6":
            # Opção 6 - Logout
            print("\n\n\t\t\tSESSAO TERMINADA")
            print("\t\t\tSERÁ REDIRECIONADO PARA O MENU INICIAL...")
            time.sleep(2)
            break
        else:
            print("Opção inválida. Tente novamente.")
            print("\t\t\tPressione qualquer tecla para continuar...")
            msvcrt.getch()


###################################################
########## FUNÇÕES DO MENU ADMINISTRADOR ##########
###################################################
###################################################

####################  OPCAO 1 #####################
def gerir_viagens():
    while True:
        # Limpa o terminal
        os.system('cls')

        print("\t\t\tGESTÃO DE VIAGENS")
        print("\t\t\t1) Adicionar nova viagem")
        print("\t\t\t2) Alterar viagem existente")
        print("\t\t\t3) Apagar viagem existente")
        print("\t\t\t4) Consultar detalhes de viagens")
        print("\t\t\t5) Voltar ao Menu Administrador\n")
        
        option = input("\t\t\tSelecione uma opção: ")

        if option == "1":
            adicionar_viagem()

        elif option == "2":
            alterar_viagem()

        elif option == "3":
            eliminar_viagem()

        elif option == "4":
            consulta_viagem_admin()

        elif option == "5":
            return
        
        else:
            print("\n\t\t\tOpção inválida. Tente novamente.")
            print("\t\t\tPressione qualquer tecla para continuar...")
            msvcrt.getch()


##########  FUNÇOES AUXILIARES - OPCAO 1 ##########

def adicionar_viagem():
    while True:
        # Limpa o terminal
        os.system('cls')

        #Obter as informações da nova viagem
        print("\t\t\t------ INTRODUZA OS DADOS DA NOVA VIAGEM ------\n")


        #PARTIDA E CHEGADA
        partida = input("\t\t\tLocal da partida: ")

        #Só devem existir viagens de ou para coimbra
        if (partida.lower() == "coimbra"):
            destino = input("\t\t\tDestino: ")

            #O destino deve ser diferente da partida
            if (partida.lower() == destino.lower()):
                print("\n\t\t\tO local de destino deve ser diferente do de partida.")
                print("\t\t\tPressione qualquer tecla para introduzir novos dados...")
                msvcrt.getch()
                continue
        else: 
            print("\t\t\tDestino: Coimbra")
            destino = "Coimbra"

        #HORARIOS E DURAÇAO
        data_partida_str = input("\t\t\tData de Partida (AAAA-MM-DD HH:MM:SS): ")
        data_chegada_str = input("\t\t\tData de Chegada (AAAA-MM-DD HH:MM:SS): ")
        
        data_partida = datetime.strptime(data_partida_str, "%Y-%m-%d %H:%M:%S")
        data_chegada = datetime.strptime(data_chegada_str, "%Y-%m-%d %H:%M:%S")
        
        # verificar se a data de partida é no futuro e se a data de chegada é posterior à data de partida
        if data_partida < datetime.now():
            print("\n\t\t\tA data de partida tem que ser no futuro.")
            print("\t\t\tPressione qualquer tecla para introduzir novos dados...")
            msvcrt.getch()
            continue

        elif data_chegada <= data_partida:
            print("\n\t\t\tA data de chegada tem que ser posterior à data de partida.")
            print("\t\t\tPressione qualquer tecla para introduzir novos dados...")
            msvcrt.getch()
            continue
        
        # calcular a duração da viagem em minutos
        duracao = int((data_chegada - data_partida).total_seconds() // 60)
        print("\t\t\tDuração (em minutos): {0}". format(duracao))

        #PREÇO E DISTANCIA
        preco_base = input("\t\t\tPreço Base (em euros): ")
        distancia = input("\t\t\tDistância (em km): ")
        

        #AUTOCARRO
        print("\n\t\t\tDados introduzidos com sucesso!")
        print("\t\t\tPressione qualquer tecla para proceder à escolha do autocarro associado a esta viagem...")
        msvcrt.getch()
        os.system('cls')

        # Listar os autocarros disponíveis para seleção
        cur.execute("SELECT matricula, marca, modelo, lugares FROM autocarro")
        autocarros = cur.fetchall()

        print("\t\t\t-------LISTA DE AUTOCARROS COIMBRA BUS --------")
        #Exibicao dos autocarros disponíveis numa tabela 
        headers = ["Matricula", "Marca", "Modelo", "Número de Lugares", "Índice"]
        table = []
        indice = 1

        for autocarro in autocarros:
            table.append([autocarro[0], autocarro[1], autocarro[2], autocarro[3], indice])
            indice += 1

        print(tabulate(table, headers=headers, tablefmt="grid"), "\n")

        escolha=input("\n\t\t\tSelecione o índice do autocarro pretendido: ")

        if escolha.isdigit() and int(escolha) in range(1, len(autocarros)+1):
            #Determinar cliente escolhido
            autocarro = autocarros[int(escolha)-1]

            lugares = autocarro['lugares']
            matricula = autocarro['matricula']
        
        else: #Se o cliente ja tiver estatuto Gold
            print("\n\t\t\tOpção inválida!")
            print("\t\t\tPressione qualquer tecla para introduzir novos dados...")
            msvcrt.getch()
            continue
    
        
        # Inserir a nova viagem na base de dados
        cur.execute("INSERT INTO viagem (partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado, autocarro_matricula) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, 'por realizar', %s)", (partida, destino, data_partida_str, data_chegada_str, duracao, lugares, preco_base, distancia, matricula))
        conn.commit()


        # Confirmar a inserção na base de dados
        print("\n\n\t\t\tViagem adicionada com sucesso!")
        print("\t\t\tPressione qualquer tecla para continuar...")
        msvcrt.getch()
        return


def eliminar_viagem():
    # Limpa o terminal
    os.system('cls')

    # Consultar a base de dados e obter todas as viagens com estado "por realizar"
    if (mostra_viagens("por realizar") == 0):
        print("\t\t\tNão existem viagens por realizar.")
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu de Gestão de Viagens...")
        msvcrt.getch()
        return

    # Pedir ao administrador que selecione uma viagem pelo ID_viagem
    id_viagem = input("\n\n\t\t\tIntroduza o ID da viagem que deseja apagar: ")

    # Verificar se a viagem existe e se o estado ainda é "por realizar"
    cur.execute("SELECT * FROM viagem WHERE id_viagem_= %s AND estado = 'por realizar'", (id_viagem,))
    viagem = cur.fetchone()

    if not viagem:
        print("\n\t\t\tA viagem selecionada não existe ou já não pode ser cancelada.")
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu de Gestão de Viagens...")
        msvcrt.getch()
        return
    else:
        #Ainda temos de verificar se existe alguma reserva associada a essa viagem
        cur.execute('SELECT * FROM reserva WHERE viagem_id_viagem_ = %s', (id_viagem,))
        reservas = cur.fetchall()

        if reservas:
            print("\n\t\t\tA viagem selecionada já tem reservas associadas e não pode ser eliminada.")
            print("\n\t\t\tPressione qualquer tecla para regressar ao Menu de Gestão de Viagens...")
            msvcrt.getch()
            return


    # Caso a viagem exista e o estado ainda seja "por realizar", apagar a viagem da base de dados
    # Eliminar primeiro as alterações de viagem associadas à viagem a eliminar
    cur.execute("DELETE FROM alteracao_viagem WHERE viagem_id_viagem_ = %s", (id_viagem,))
    conn.commit()
    cur.execute("DELETE FROM viagem WHERE id_viagem_=%s", (id_viagem,))
    conn.commit()

    #Regressar ao Menu de Gestão de Viagens
    print(f"\n\t\t\tA viagem com ID {id_viagem} foi apagada com sucesso.")
    print("\n\t\t\tPressione qualquer tecla para regressar ao Menu de Gestão de Viagens...")
    msvcrt.getch()


def alterar_viagem():
    # Limpa o terminal
    os.system('cls')

    # Consultar a base de dados e obter todas as viagens com estado "por realizar"
    if (mostra_viagens("por realizar") == 0):
        print("\t\t\tNão existem viagens por realizar.")
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu de Gestão de Viagens...")
        msvcrt.getch()
        return

    # Solicitar o ID da viagem que será alterada
    id_viagem = input("\n\n\t\t\tIntroduza o ID da viagem cujo preço deseja alterar: ")

    #Mostrar a viagem escolhida
    i = mostra_viagem(id_viagem) 

    if (i == 0):
        print("\n\t\t\tA viagem selecionada não existe ou já foi realizada.")
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu de Gestão de Viagens...")
        msvcrt.getch()
        return

    # Solicitar o novo preço da viagem
    novo_preco = input("\n\t\t\tIntroduza o novo preço da viagem (em euros): ")

    # Obter o preço antigo e inserir o registro de alteração na tabela alteracao_viagem
    cur.execute("SELECT preco_base_ FROM viagem WHERE id_viagem_ = %s ", (id_viagem,))
    preco_antigo = cur.fetchone()[0]


    #Inserir a nova alteração no historico
    cur.execute("INSERT INTO alteracao_viagem (data, preco_antigo, viagem_id_viagem_) VALUES (DATE_TRUNC('minute', CURRENT_TIMESTAMP), %s, %s)", (preco_antigo, id_viagem))
    conn.commit()

    # Atualizar o preço na tabela viagem
    cur.execute("UPDATE viagem SET preco_base_ = %s WHERE id_viagem_ = %s", (novo_preco, id_viagem))
    conn.commit()

    print("\n\t\t\tO preço da viagem foi atualizado com sucesso.")
    print("\t\t\tPressione qualquer tecla para regressar ao Menu de Gestão de Viagens...")
    msvcrt.getch()
    return


def consulta_viagem_admin():
    while True:
        # Limpa o terminal
        os.system('cls')

        print("\t\t\t------ CONSULTA DE VIAGENS -----\t\t\t")
        print("\t\t\t1) Viagens por realizar")
        print("\t\t\t2) Viagens a decorrer")
        print("\t\t\t3) Viagens já efetuadas")
        print("\t\t\t4) Sair")
        
        option = input("\t\t\tSelecione a opção pretendida: ")

        #Determinar o tipo de viagem que se pretende mostrar ou regressar ao menu anterior
        if option == "1":
            estadov = "por realizar"

        elif option == "2":
            estadov = "a decorrer"
            
        elif option == "3":
            estadov = "já efetuada"

        elif option == "4":
            return
        
        else:
            print("Opção inválida. Tente novamente.")
            print("\t\t\tPressione qualquer tecla para continuar...")
            msvcrt.getch()
            continue

        #Mostrar viagens consoante a opçao escolhida
        os.system('cls')
        if (mostra_viagens(estadov) == 0):
            print("\n\n\t\t\tNão existem viagens do tipo '", estadov , "'.")
        else:
            detail = input("\n\n\t\t\tPretende ver os detalhes de alguma viagem? [S/N]: ")

            if (detail.lower() == "s"): #qualquer outra resposta será considerado como 'não'
                # Pedir ao administrador que selecione uma viagem pelo ID_viagem
                id_viagem = input("\t\t\tIntroduza o ID da viagem cujos detalhes deseja visualizar: ")

                #Mostrar a viagem escolhida

                i = mostra_viagem(id_viagem)

                if (i == 0):
                    print("\n\n\t\t\tOpção inválida")

        #Serve apenas para esperar que o utilizador introduza alguma tecla antes de voltar a exibir o Menu
        print("\t\t\tPressione qualquer tecla para regressar ao Menu Consulta de Viagens...")
        msvcrt.getch()



    


####################  OPCAO 2 #####################
def gerir_autocarros():
    while True:
        # Limpa o terminal
        os.system('cls')

        print("\t\t\tGESTÃO DE AUTOCARROS")
        print("\t\t\t1) Adicionar novo autocarro")
        print("\t\t\t2) Consultar lista de autocarros")
        print("\t\t\t3) Voltar ao Menu Administrador\n")
        
        option = input("\t\t\tSelecione uma opção: ")

        if option == "1":
            adicionar_autocarro()

        elif option == "2":
            consulta_autocarros()

        elif option == "3":
            return
        
        else:
            print("\n\t\t\tOpção inválida. Tente novamente.")
            print("\t\t\tPressione qualquer tecla para continuar...")
            msvcrt.getch()
        
    



##########  FUNÇOES AUXILIARES - OPCAO 2 ##########

def adicionar_autocarro():
    while True:
        # Limpa o terminal
        os.system('cls')

        # Obter as informações do novo autocarro
        print("\t\t\t------ INTRODUZA OS DADOS DO NOVO AUTOCARRO ------\n")

        matricula = input("\t\t\tMatrícula: ")

        # Verificar se já existe um autocarro com a mesma matrícula
        cur.execute("SELECT * FROM autocarro WHERE matricula = %s", (matricula,))
        autocarro_existente = cur.fetchone()
        if autocarro_existente:
            print("\n\t\t\tEsse autocarro já foi adicionado à frota")
            print("\t\t\tPressione qualquer tecla para inserir novos dados...")
            msvcrt.getch()
            continue

        marca = input("\t\t\tMarca: ")
        modelo = input("\t\t\tModelo: ")
        lugares = input("\t\t\tNº de lugares (entre 10 e 50): ")
        
        # Verificar se o número de lugares é um inteiro entre 10 e 50
        try:
            lugares = int(lugares)
            if lugares < 10 or lugares > 50:
                print("\n\t\t\tO número de lugares deve ser entre 10 e 50!")
                print("\t\t\tPressione qualquer tecla para inserir novos dados...")
                msvcrt.getch()
                continue
        except ValueError:
            print("\n\t\t\tO número de lugares deve ser um inteiro!")
            print("\t\t\tPressione qualquer tecla para inserir novos dados...")
            msvcrt.getch()
            continue
        

        # Inserir o novo autocarro na base de dados
        cur.execute("INSERT INTO autocarro (matricula, marca, modelo, lugares) VALUES (%s, %s, %s, %s)", (matricula, marca, modelo, lugares))
        conn.commit()
        
        print("\n\t\t\tAutocarro adicionado com sucesso!")
        print("\t\t\tPressione qualquer tecla para regressar ao Menu Administrador...")
        msvcrt.getch()
        return


def consulta_autocarros():
    os.system('cls')

    #Obter lista de todos os autocarros
    cur.execute("SELECT matricula, marca, modelo, lugares from autocarro")
    autocarros=cur.fetchall()

    # Verificar se há autocarros a apresentar
    if not autocarros: #Isto nunca deverá acontecer, uma vez que sem autocarros é impossivel adicionar viagens
        print("\t\t\tNão existe qualquer autocarro registado")
    else:
        print("\t\t\t-------LISTA DE AUTOCARROS COIMBRA BUS --------")
        #Exibicao dos autocarros disponíveis numa tabela 
        headers = ["Matricula", "Marca", "Modelo", "Número de Lugares"]
        table = []

        for autocarro in autocarros:
            table.append([autocarro[0], autocarro[1], autocarro[2], autocarro[3]])

        print(tabulate(table, headers=headers, tablefmt="grid"), "\n")
        
        
    print("\t\t\tPressione qualquer tecla para regressar ao Menu de Gestão de Autocarros...")
    msvcrt.getch()
    return




####################  OPCAO 3 #####################
def mensagem_admin(admin):
    while True:
        # Limpa o terminal
        os.system('cls')

        #Obter a lista de todos os clientes
        cur.execute("SELECT nome_completo, nif, telefone, email, tipo_gold from cliente")
        clientes=cur.fetchall()

        #Para o caso de não exisitirem clientes registados ainda, regressamos ao menu anterior
        if not clientes:
            print("\t\t\tNão existe qualquer cliente registado de momento")
            print("\t\t\tPressione qualquer tecla para regressar ao Menu Administrador...")
            msvcrt.getch()
            return

        #MENU DE ENVIO DE MENSAGENS
        print("\t\t\t ------ ENVIO DE MENSAGENS -------")
        print("\t\t\t1) Enviar mensagem a um cliente")
        print("\t\t\t2) Enviar mensagem para todos os clientes")
        print("\t\t\t3) Voltar ao Menu Administrador\n")
        
        option = input("\n\t\t\tSelecione uma opção: ")

        if option == "1": 
            message2client(clientes, admin) 
            #Funçao que permite escolher ao admin escolher um cliente
            #da lista de clientes e enviar-lhe mensagem
            
        elif option == "2":
            message2all(clientes, admin) 
            #Funçao que permite escolher ao admin enviar
            # mensagem a todos os clientes

        elif option == "3":
            return
        
        else:
            print("\n\t\t\tOpção inválida. Tente novamente.")
            print("\t\t\tPressione qualquer tecla para continuar...")
            msvcrt.getch()
            continue





####################  OPCAO 4 #####################
def gerir_clientes():
    
    os.system('cls')

    #Obter a lista de todos os clientes
    cur.execute("SELECT nome_completo, nif, telefone, email, tipo_gold from cliente")
    clientes=cur.fetchall()

    #Para o caso de não exisitirem clientes registados ainda
    if not clientes:
        print("\t\t\tNão existe qualquer cliente registado de momento")
        print("\t\t\tPressione qualquer tecla para regressar ao Menu Administrador...")
        msvcrt.getch()
        return

    #Exibicao dos clientes registados numa tabela 
    headers = ["Nome", "NIF", "Telemovel", "Email", "Estatuto Gold" , "Indice do Cliente"]
    table = []
    indice = 1

    for atributo_cliente in clientes:
        table.append([atributo_cliente[0], atributo_cliente[1], atributo_cliente[2], atributo_cliente[3], atributo_cliente[4], indice])
        indice += 1
        
    print("\t\t\t-------LISTA DE CLIENTES COIMBRA BUS --------")
    print("\n")
    print(tabulate(table, headers=headers))
    
    print("\n\n\n\t\t\t-------OPÇÕES -------")
    print("\t\t\t1) Atribuir estatuto Gold a um cliente")
    print("\t\t\t2) Retirar estatuto Gold a um cliente")
    print("\t\t\t3) Regressar ao Menu administrador")

    opcao = input("\n\t\t\tSelecione uma das opções: ")


    if opcao == '1':
        escolha=input("\n\t\t\tSelecione o índice do cliente ao qual pretende atribuir o estatuto Gold: ")

        if escolha.isdigit() and int(escolha) in range(1, len(clientes)+1):
                #Determinar cliente escolhido
                cliente_escolhido = clientes[int(escolha)-1]
                
                #Se o cliente ainda nao tiver estatuto Gold
                if(cliente_escolhido[4]==False):
                    cur.execute("UPDATE cliente SET tipo_gold = 'True' WHERE nif = %s", (cliente_escolhido[1],))
                    conn.commit()
                    print(f"\n\t\t\tFoi atribuído com sucesso ao cliente {atributo_cliente[0]} o estatuto Gold.")
                    print("\t\t\tSelecione qualquer tecla para ser redirecionado para o Menu Administrador...")
                    msvcrt.getch()
                    return 
        
                else: #Se o cliente ja tiver estatuto Gold
                    print("\n\t\t\tO cliente que selecionou já tem estatuto Gold!")
                    print("\t\t\tSelecione qualquer tecla para ser redirecionado para o Menu Administrador...")
                    msvcrt.getch()
                    return
        else:
            print("\n\t\t\tOpção inválida!")
            print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Administrador...")
            msvcrt.getch()
            return


    elif opcao == '2':
        escolha=input("\t\t\tSelecione o índice do cliente ao qual pretende retirar o estatuto Gold: ")

        if escolha.isdigit() and int(escolha) in range(1, len(clientes)+1):
            #Determinar cliente escolhido
            cliente_escolhido = clientes[int(escolha)-1]
    
            #Se o cliente ja tiver estatuto Gold
            if(cliente_escolhido[4]==True):
                cur.execute("UPDATE cliente SET tipo_gold = 'False' WHERE nif = %s", (cliente_escolhido[1],))
                conn.commit()
                print(f"\n\t\t\tFoi retirado com sucesso ao cliente {atributo_cliente[0]} o estatuto Gold.")
                print("\t\t\tSelecione qualquer tecla para ser redirecionado para o Menu Administrador...")
                msvcrt.getch()
                return 
            
            else: #Se o cliente ainda nao tiver estatuto Gold
                print("\n\t\t\tO cliente que selecionou não tinha estatuto Gold!")
                print("\t\t\tSelecione qualquer tecla para ser redirecionado para o Menu Administrador...")
                msvcrt.getch()
                return
        else:
            print("\n\t\t\tOpção inválida!")
            print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Administrador...")
            msvcrt.getch()
            return       

    elif opcao == '3':
        print("\n\n\t\t\tSerá redirecionado para o Menu Administrador...")
        time.sleep(2)
        return

    else:
        print("\n\t\t\tOpção inválida!")
        print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Administrador...")
        msvcrt.getch()
        return            




####################  OPCAO 5 #####################

def ver_estatisticas():
    print("\t\t\tOPCOES:")
    print("\t\t\t1) Viagem mais vendida num determinado mês")
    print("\t\t\t2) O cliente que mais viagens comprou num determinado mês")
    print("\t\t\t3) viagens que não tiveram reservas num determinado mês")
    print("\t\t\t4) Reservas de uma determinada viagem")
    print("\t\t\t5) Reservas canceladas de uma determinada viagem")
    print("\t\t\t6) Reservas/clientes em espera")
    print("\t\t\t7) Percurso com mais clientes num determinado mês")
    print("\t\t\t8) Volume de vendas em cada mês")
    print("\t\t\t9) Dia do ano em que houve mais vendas")
    print("\t\t\t10) Voltar ao Menu do Administrador")

    escolha=input("Selecione a estatistica que pretende visualizar: ")

    #VIAGEM MAIS VENDIDA
    if escolha == '1':
        #Verificacao do mes:
        while True:
            mes_selecionado = input("\n\t\t\tPretende visualizar a viagem mais vendida relativa a que mes: ")
            if mes_selecionado.isdigit() and 0 < int(mes_selecionado) <= 12:
                break
            print("Por favor, insira um mês válido (1-12)")
        #Consulta em SQL
        cur.execute("SELECT viagem.id_viagem_, viagem.partida, viagem.destino, viagem.data_de_partida, viagem.data_de_chegada , COUNT(reserva.id_reserva_) AS num_reservas \
             FROM viagem JOIN reserva ON viagem.id_viagem_ = reserva.viagem_id_viagem_\
             WHERE EXTRACT(MONTH FROM reserva.data) = %s\
             GROUP BY viagem.id_viagem_\
             ORDER BY num_reservas DESC\
             LIMIT 1;", (mes_selecionado,))
        viagem_mais_vendida = cur.fetchall()
        #Exibicao da viagem em questao numa tabela
        headers = ["ID Viagem", "Local de Partida", "Local de Chegada", "Data de Partida", "Data de Chegada", "Numero de Reservas"]
        table = []
        for atributo_mais in viagem_mais_vendida:
            table.append([atributo_mais[0], atributo_mais[1], atributo_mais[2], atributo_mais[3], atributo_mais[4], atributo_mais[5]])
        print("\n\n")
        print(tabulate(table, headers=headers))
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
        msvcrt.getch()
        return
    
    #CLIENTE QUE MAIS COMPROU VIAGENS
    elif escolha == '2':
        #Verificacao do mes:
        while True:
            mes_selecionado = input("\n\t\t\tPretende visualizar o cliente que mais comprou viagens de que mes: ")
            if mes_selecionado.isdigit() and 0 < int(mes_selecionado) <= 12:
                break
            print("Por favor, insira um mês válido (1-12)")
            #consulta em sql
        cur.execute("SELECT reserva.cliente_pessoa_nif, cliente.nome_completo, cliente.email, cliente.telefone, COUNT(reserva.id_reserva_) AS num_reservas \
                    FROM reserva JOIN cliente ON reserva.cliente_pessoa_nif = cliente.nif\
                    WHERE MONTH(reserva.data) = %s\
                    GROUP BY reserva.cliente_pessoa_nif\
                    ORDER BY num_reservas DESC\
                    LIMIT 1;", (mes_selecionado,))
        cliente_mais_compras = cur.fetchall()
        headers = ["NIF", "Nome", "Email", "Telemovel"]
        table = []
        for atributo_mais in cliente_mais_compras:
            table.append([atributo_mais[0], atributo_mais[1], atributo_mais[2], atributo_mais[3]])
        print("\n\n")
        print(tabulate(table, headers=headers))
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
        msvcrt.getch()
        return
    
    #VIAGENS QUE NAO TIVERAM RESERVAS
    elif escolha == '3':
        while True:
            mes_selecionado = input("\n\t\t\tPretende visualizar as viagens que nao tiveram reservas de que mes: ")
            if mes_selecionado.isdigit() and 0 < int(mes_selecionado) <= 12:
                break
            print("Por favor, insira um mês válido (1-12)")
        
        cur.execute("SELECT viagem.id_viagem_, viagem.partida, viagem.destino, viagem.data_de_partida, viagem.data_de_chegada \
                    FROM viagem\
                    LEFT JOIN reserva ON viagem.id_viagem_ = reserva.viagem_id_viagem_ AND MONTH(reserva.data) = %s\
                    WHERE reserva.id_reserva_ IS NULL;", (mes_selecionado,))
        #Usamos um left JOIN para serem selecionadas todas as viagens independentemente de terem ou nao reservas associadas 
        viagem_mais_vendida = cur.fetchall()
        #Exibicao atraves de uma tabela
        headers = ["ID Viagem", "Local de Partida", "Local de Chegada", "Data de Partida", "Data de Chegada", "Numero de Reservas"]
        table = []
        for atributo_mais in viagem_mais_vendida:
            table.append([atributo_mais[0], atributo_mais[1], atributo_mais[2], atributo_mais[3], atributo_mais[4], atributo_mais[5]])
        print("\n\n")
        print(tabulate(table, headers=headers))
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
        msvcrt.getch()
        return
    
    #Nº DE RESERVAS DE UMA DETERMINADA VIAGEM
    elif escolha == '4':
        cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado FROM viagem ")
        viagens = cur.fetchall()
        print("\t\t\tPressione qualquer tecla para continuar...")
        msvcrt.getch()
            
        #se não houver viagens disponíveis 
        if not viagens:
            print("\n\t\t\tNão há viagens disponíveis com esses filtros de pesquisa.")
            print("\t\t\tPressione qualquer tecla para regressar ao Menu do Administrador...")
            msvcrt.getch()
            return   #retorna à função menu cliente
    
        #adicionar à tabela mais uma coluna com o indice da viagem que vai permitir que o utilizador, através do exibição da tabela, escolha a viagem que mais lhe agrade 
        headers = ["ID Viagem", "Partida", "Destino", "Data de Partida", "Data de Chegada", "Duração", "Lugares Disponíveis", "Distância", "Preço", "Estado", "Indice da viagem"]
        table = []
        indice = 1
    
        for atributo_viagem in viagens:
            table.append([atributo_viagem[0], atributo_viagem[1], atributo_viagem[2], atributo_viagem[3], atributo_viagem[4], atributo_viagem[5], atributo_viagem[6], atributo_viagem[7], atributo_viagem[8], atributo_viagem[9], indice])
            indice += 1
        #Exibicao, primeiramente, de uma tabela com todas as viagens disponiveis para escolher  
        print("\n\n")
        print(tabulate(table, headers=headers))
        
        while True:
            option = input("\n\t\t\tPretende visualizar o numero de reservas de que viagem? Digite o indice: ")
            if option.isdigit() and int(option) in range(1, len(viagens)+1):
                break
            print("\t\t\tPor favor, insira um indice valido")
        viagem_escolhida = viagens[int(option)-1]
        #Depois de escolhida a viagem pretendida, iremos fazer uma consulta em sql para visualizar quantas reservas estao associadadas à viagem escolhida previamente
        cur.execute("SELECT reserva.id_reserva_, reserva.data, reserva.valor_pago, reserva.lugar_, reserva.cliente_pessoa_nif, cliente.nome_completo \
                    FROM reserva JOIN cliente ON reserva.cliente_pessoa_nif = cliente.nif \
                    WHERE reserva.viagem_id_viagem_ = %s", (viagem_escolhida[0]))
        reservas=cur.fetchall()
        #Mostrar as reservas para uma determinada viagem escolhida previamente numa Tabela
        headers = ["ID Reserva", "Data da reserva", "Valor Pago", "Lugar", "NIF", "Nome Completo"]
        table = [] 
        for atributo in reservas:
            table.append([atributo[0], atributo[1], atributo[2], atributo[3], atributo[4], atributo[5]])              
        print("\n\n")
        print(tabulate(table, headers=headers))
        print("\n\t\t\tEstas sao todas as reservas correspondentes a viagem selecionada")
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu do Administrador...")
        msvcrt.getch()
        return
    
    #RESERVAS CANCELADAS DE UMA DETERMINADA VIAGEM
    #Fizemos o mesmo raciocinio da opção anterior
    #So acrescentamos o estado = cancelada à reserva associada a viagem escolhida
    elif escolha == '5':
        
        cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado FROM viagem ")
        viagens = cur.fetchall()
        print("\t\t\tPressione qualquer tecla para continuar...")
        msvcrt.getch()
            
        #se não houver viagens disponíveis 
        if not viagens:
            print("\n\t\t\tNão há viagens disponíveis com esses filtros de pesquisa.")
            print("\t\t\tPressione qualquer tecla para regressar ao Menu do Administrador...")
            msvcrt.getch()
            return   #retorna à função menu cliente
    
        #adicionar à tabela mais uma coluna com o indice da viagem que vai permitir que o utilizador, através do exibição da tabela, escolha a viagem que mais lhe agrade 
        headers = ["ID Viagem", "Partida", "Destino", "Data de Partida", "Data de Chegada", "Duração", "Lugares Disponíveis", "Distância", "Preço", "Estado", "Indice da viagem"]
        table = []
        indice = 1

    
        for atributo_viagem in viagens:
            table.append([atributo_viagem[0], atributo_viagem[1], atributo_viagem[2], atributo_viagem[3], atributo_viagem[4], atributo_viagem[5], atributo_viagem[6], atributo_viagem[7], atributo_viagem[8], atributo_viagem[9], indice])
            indice += 1
                
        print("\n\n")
        print(tabulate(table, headers=headers))
        
        while True:
            option = input("\n\t\t\tPretende visualizar o numero de reservas canceladas de que viagem? Digite o indice: ")
            if option.isdigit() and int(option) in range(1, len(viagens)+1):
                break
            print("\t\t\tPor favor, insira um indice valido")
        viagem_escolhida = viagens[int(option)-1]
        cur.execute("SELECT reserva.id_reserva_, reserva.data, reserva.valor_pago, reserva.lugar_, reserva.cliente_pessoa_nif, cliente.nome_completo \
                    FROM reserva JOIN cliente ON reserva.cliente_pessoa_nif = cliente.nif \
                    WHERE reserva.viagem_id_viagem_ = %s AND reserva.estado='cancelada'", (viagem_escolhida[0]))
        reservas=cur.fetchall()
        #Mostrar as reservas para uma determinada viagem escolhida previamente
        headers = ["ID Reserva", "Data da reserva", "Valor Pago", "Lugar", "NIF", "Nome Completo"]
        table = [] 
        for atributo in reservas:
            table.append([atributo[0], atributo[1], atributo[2], atributo[3], atributo[4], atributo[5]])              
        print("\n\n")
        print(tabulate(table, headers=headers))
        print("\n\t\t\tEstas sao as reservas canceladas correspondentes a viagem selecionada")
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu do Administrador...")
        msvcrt.getch()
        return
    
    #RESERVAS/CLIENTES EM ESPERA
    elif escolha == '6':
        cur.execute("SELECT reserva.id_reserva_, reserva.data, reserva.valor_pago, reserva.lugar_, reserva.cliente_pessoa_nif, cliente.nome_completo\
                    FROM reserva JOIN cliente ON reserva.cliente_pessoa_nif = cliente.nif\
                    WHERE reserva.estado = 'pendente'")
        reservas = cur.fetchall()
        headers = ["ID Reserva", "Data da reserva", "Valor Pago", "Lugar", "NIF", "Nome Completo"]
        table = [] 
        for atributo in reservas:
            table.append([atributo[0], atributo[1], atributo[2], atributo[3], atributo[4], atributo[5]])              
        print("\n\n")
        print(tabulate(table, headers=headers))
        print("\n\t\t\tEstas sao as reservas/clientes em lista de espera")
        print("\n\t\t\tPressione qualquer tecla para regressar ao Menu do Administrador...")
        msvcrt.getch()
        return
    
    #PERCURSO COM MAIS CLIENTES NUM DETERMINADO MES 
    elif escolha == '7':
        while True:
            mes_selecionado = input("\n\t\t\tPretende visualizar o percurso com mais clientes relativo a que mes: ")
            if mes_selecionado.isdigit() and 0 < int(mes_selecionado) <= 12:
                break
            print("Por favor, insira um mês válido (1-12)")

        cur.execute("SELECT partida, destino, COUNT(reserva.id_reserva_) AS num_reservas\
                    FROM viagem JOIN reserva ON viagem.id_viagem_ = reserva.viagem_id_viagem_\
                    WHERE MONTH(reserva.data) = %s\
                    GROUP BY partida, destino\
                    ORDER BY num_reservas DESC\
                    LIMIT 1", (mes_selecionado))
        
        Percurso_mais_clientes = cur.fetchall()
        headers = ["Local de partida", "Local de Destino", "Nº de Reservas Efetuadas"]
        table = [] 
        for atributo in Percurso_mais_clientes:
            table.append([atributo[0], atributo[1], atributo[2]])              
        print("\n\n")
        print(tabulate(table, headers=headers))
        print(f"\n\t\t\tEste e o percurso com mais clientes no {mes_selecionado} mes")
        print("\t\t\tPressione qualquer tecla para regressar ao Menu do Administrador...")
        msvcrt.getch()
        return
    
    #VOLUME DE VENDAS EM CADA MÊS
    elif escolha == '8':
        cur.execute("SELECT EXTRACT(MONTH FROM reserva.data) AS mes, COUNT(reserva.id_reserva_) AS num_reservas\
        FROM reserva\
        GROUP BY mes\
        ORDER BY mes")
        resultado=cur.fetchall()
        headers = ["MES", "Nº de Reservas Efetuadas"]
        table = [] 
        for atributo in resultado:
            table.append([atributo[0], atributo[1]])              
        print("\n\n")
        print(tabulate(table, headers=headers))
        print("\n\t\t\tEste e o Volume de vendas de cada mes")
        print("\t\t\tPressione qualquer tecla para regressar ao Menu do Administrador...")
        msvcrt.getch()
        return
    
    #DIA DO ANO COM MAIS VENDAS
    elif escolha == '9':
        cur.execute("SELECT reserva.data AS dia_do_ano, COUNT(reserva.id_reserva_) AS num_reservas\
                    FROM reserva\
                    GROUP BY dia_do_ano\
                    ORDER BY num_reservas DESC\
                    LIMIT 1")
        resultado=cur.fetchall()
        headers = ["Data", "Nº de Reservas Efetuadas"]
        table = [] 
        for atributo in resultado:
            table.append([atributo[0], atributo[1]])              
        print("\n\n")
        print(tabulate(table, headers=headers))
        print("\n\t\t\tEste foi o dia do ano com mais reservas efetuadas")
        print("\t\t\tPressione qualquer tecla para regressar ao Menu do Administrador...")
        msvcrt.getch()
        return
    
    #VOLTAR AO MENU ADMINISTRADOR
    elif escolha == '10':
        return




###################################################
##########         MENU CLIENTE    ################
###################################################
###################################################

def menu_cliente(cliente):
     while True:
        # Limpa o terminal
        os.system('cls')

        nome = cliente['nome_completo']
        nif = cliente['nif']

        print(f"\t\t\tMENU CLIENTE ---- {nome}")
        print("\t\t\t1) Consultar e Reservar viagens")
        print("\t\t\t2) Filtros avancados de pesquisa de viagens")
        print("\t\t\t3) Consultar e cancelar reservas já efetuadas")
        print("\t\t\t4) Notificações")
        print("\t\t\t5) Logout")
      
        option = input("\n\t\t\tSelecione uma opção: ")
        
        if option == "1":
            # Opção 1 - Consultar e reservar viagens
            consultar_reservar_viagens(nif)  

        elif option == '2':
            # Opção 2 - Filtros avancados de pesquisa de viagens
            pesquisa_avancada()

        elif option == "3":
            # Opção 3 - Consultar e cancelar reservas já efetuadas
            gerir_reservas(nif)

        elif option == "4":
            # Opção 4 - Ver notificações
            visualizar_notificacoes(nif)
            
        elif option == "5":
            # Opção 5 - Logout
            print("\n\n\t\t\tSESSAO TERMINADA")
            print("\t\t\tSERÁ REDIRECIONADO PARA O MENU INICIAL...")
            time.sleep(2)
            break
        
        else:
            print("\n\t\t\tOpção inválida. Tente novamente.")
            print("\t\t\tPressione qualquer tecla para continuar...")
            msvcrt.getch()


###################################################
##########    FUNÇÕES DO MENU CLIENTE    ##########
###################################################
###################################################


####################  OPCAO 1 #####################


def consultar_reservar_viagens(nif1):
  
    # Limpa o terminal
    os.system('cls')

    #filtros de pesquisa 
    print("\t\t\tFILTROS DE PESQUISA\n")

    partida = input("\n\t\t\tLocal de partida: ")
    destino = input("\t\t\tLocal de chegada: ")   
    data_de_partida = input("\t\t\tData de partida (formato YYYY-MM-DD): ")

    cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                FROM viagem \
                WHERE partida = %s AND destino = %s AND data_de_partida >= %s  AND estado = 'por realizar'", (partida, destino, data_de_partida))
    viagens = cur.fetchall()       

    #se não houver viagens disponíveis 
    if not viagens:
        print("\n\t\t\tNão há viagens disponíveis com esses filtros de pesquisa.")
        print("\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
        msvcrt.getch()
        return   #retorna à função menu cliente
    
    #adicionar à tabela mais uma coluna com o indice da viagem que vai permitir que o utilizador, através do exibição da tabela, escolha a viagem que mais lhe agrade 
    headers = ["ID Viagem", "Partida", "Destino", "Data de Partida", "Data de Chegada", "Duração", "Lugares Disponíveis", "Distância", "Preço", "Estado", "Indice da viagem"]
    table = []
    indice = 1

    for atributo_viagem in viagens:
        table.append([atributo_viagem[0], atributo_viagem[1], atributo_viagem[2], atributo_viagem[3], atributo_viagem[4], atributo_viagem[5], atributo_viagem[6], atributo_viagem[7], atributo_viagem[8], atributo_viagem[9], indice])
        indice += 1
        
    #Exibir a tabela das viagens disponíveis 
    print("\n\n")
    print(tabulate(table, headers=headers))

    #Associar a uma variavel o estauto do cliente
    cur.execute("SELECT tipo_gold from cliente WHERE nif=%s", str(nif1))
    estatuto=cur.fetchone()["tipo_gold"]
    
    #Reservar uma viagem
    print("\n\t\t\tOPÇÕES:")
    print("\t\t\t1) Reservar uma viagem")
    print("\t\t\t2) Voltar ao menu do cliente")
    option = input("\n\t\t\tSelecione uma opcao: ")

    if option == "1":
        escolha = input("\n\t\t\tDigite o indice da viagem que deseja reservar: ")
        
        if escolha.isdigit() and int(escolha) in range(1, len(viagens)+1):

            #se já não houver mais lugares disponiveis na viagem a reserva fica pendente - RESERVA PENDENTE
            if(atributo_viagem[6]=="0"):
                viagem_escolhida = viagens[int(escolha)-1]

                cur.execute("INSERT INTO reserva  (estado, data, valor_pago, viagem_id_viagem_,cliente_pessoa_nif) VALUES ('pendente', 'DATE_TRUNC('minute', CURRENT_TIMESTAMP)','0', %s, %s)"( viagem_escolhida[0], nif1))
                print("\n\t\t\tA sua reserva está pendente, o autocarro encontra-se cheio! Enviaremos uma notificacao caso seja disponibilizado um lugar!")
                conn.commit()
                 # (-----------) #ENVIAR NOTIFICAÇAO 

                print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
                msvcrt.getch()
                return

            # Se ainda houver lugares - RESERVA ATIVA
            else:
                viagem_escolhida = viagens[int(escolha)-1]

                #Vai associar à variavél "lugar" o lugar do cliente na reserva 
                lugar = lugar_associado(viagem_escolhida[0])

                #Se o cliente for do tipo Gold tem direito a 10% de desconto
                if(estatuto == True):
                    cur.execute("INSERT INTO reserva (id_reserva_, estado, data, valor_pago, lugar_, viagem_id_viagem_, cliente_pessoa_nif) \
                                VALUES ('ativa', 'DATE_TRUNC('minute', CURRENT_TIMESTAMP)', %s, %s, %s, %s)" \
                                , (viagem_escolhida[8]*0.90, lugar, viagem_escolhida[0], nif1))
                    conn.commit()
                
                #Se o cliente nao for do tipo Gold nao tem desconto
                else:
                    cur.execute("INSERT INTO reserva (id_reserva_, estado, data, valor_pago, lugar_, viagem_id_viagem_, cliente_pessoa_nif) \
                                VALUES ('ativa', 'DATE_TRUNC('minute', CURRENT_TIMESTAMP)','2023-09-12', %s, %s, %s, %s)"\
                                , (viagem_escolhida[8], lugar, viagem_escolhida[0], nif1))
                    conn.commit()
                        
                print("\n\t\t\tRESERVA EFETUADA COM SUCESSO!\n")

                #decrementa 1 do atributo lugares disponiveis
                cur.execute("SELECT lugares_disponiveis FROM viagem WHERE id_viagem_ = %s", (viagem_escolhida[5],))
                lugares_disponiveis = cur.fetchone()["lugares_disponiveis"]
                novo_lugares_disponiveis = lugares_disponiveis - 1
                cur.execute("UPDATE viagem SET lugares_disponiveis = %s WHERE id_viagem_ = %s", (novo_lugares_disponiveis, viagem_escolhida[5]))
                conn.commit()
                print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
                msvcrt.getch()
                return
        
        else:
            print("\n\t\t\tOpção inválida!")
            print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
            msvcrt.getch()
            return
                
    elif option  == "2":
        return
        
    else:
        print("\n\t\t\tOpção inválida!")
        print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
        msvcrt.getch()
        return
            
####################  OPCAO 2 #####################          


def pesquisa_avancada():

  #Limpar o terminal 
  os.system('cls')

  #Perguntar filtos de pesquisa
  print("\n\t\t\tPRETENDE FILTRAR VIAGENS POR:")
  print("\t\t\t1) Data de Partida")
  print("\t\t\t2) Duracao") 
  print("\t\t\t3) Distancia")
  print("\t\t\t4) Nao fitrar as viagens")
  filtros = input("\t\t\tEscolha uma das opcoes: ")  

#FILTRAR POR DATA DE PARTIDA
  if filtros == '1':
      
      os.system('cls')
      data = input("\t\t\tData de partida da Viagem: ")

      #OPCAO PARA ORDENAR AS VIAGENS
      print("\n\t\t\tOPCOES:")
      print("\t\t\t1) Ordenar as viagens disponiveis por duracao")
      print("\t\t\t2) Ordenar as viagens disponiveis por data de partida") 
      print("\t\t\t3) Ordenar viagens disponiveis por distancia de Coimbra")
      print("\t\t\t4) Nao ordenar as viagens disponiveis")
      ordenacao = input("\t\t\tEscolha uma das opcoes: ")

      # Consultar viagens disponíveis tendo em conta a opcao do cliente
      if ordenacao =='1':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado\
                       FROM viagem WHERE data_de_partida >= %s  AND estado = 'por realizar' ORDER BY duracao ASC", (data))
          viagens = cur.fetchall()
      elif ordenacao == '2':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE data_de_partida >= %s  AND estado = 'por realizar'  ORDER BY data_de_partida ASC", (data))
          viagens = cur.fetchall()
      elif ordenacao == '3':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE data_de_partida >= %s AND estado = 'por realizar' ORDER BY distancia ASC", (data))
          viagens = cur.fetchall()
      elif ordenacao == '4':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE data_de_partida >= %s AND estado = 'por realizar' ", (data))
          viagens = cur.fetchall()
          #Se o utilizador inserir um caracter invalido assumimos que nao quer ordenar as suas viagens
      else:
          print("\n\t\t\tOpção inválida! As viagens nao serao ordenadas")
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE data_de_partida >= %s AND estado = 'por realizar' ", (data))
          viagens = cur.fetchall()
          time.sleep(3)
    
      #se não houver viagens disponíveis 
      if not viagens:
          print("\n\t\t\tNão há viagens disponíveis com esses filtros de pesquisa.")
          print("\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
          msvcrt.getch()
          return   #retorna à função menu cliente
     
      headers = ["ID Viagem", "Partida", "Destino", "Data de Partida", "Data de Chegada", "Duração", "Lugares Disponíveis", "Distância", "Preço", "Estado"]
      table = []
      

      for atributo_viagem in viagens:
          table.append([atributo_viagem[0], atributo_viagem[1], atributo_viagem[2], atributo_viagem[3], atributo_viagem[4], atributo_viagem[5], atributo_viagem[6], atributo_viagem[7], atributo_viagem[8], atributo_viagem[9]])
          
      #Exibir a tabela tendo em conta os filtros de pesquisa do cliente       
      print("\n\n")
      print(tabulate(table, headers=headers))
      print("\n\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
      msvcrt.getch()
      return 
  
  #FILTRAR POR DURACAO
  elif filtros == '2':
      os.system('cls')
      duracao = input("\t\t\tDuracao da Viagem: ")
      #OPCAO PARA ORDENAR AS VIAGENS
      print("\n\t\t\tOPCOES:")
      print("\t\t\t1) Ordenar as viagens disponiveis por duracao")
      print("\t\t\t2) Ordenar as viagens disponiveis por data e hora") 
      print("\t\t\t3) Ordenar viagens disponiveis por distancia de Coimbra")
      print("\t\t\t4) Nao ordenar as viagens disponiveis")
      ordenacao = input("\t\t\tEscolha uma das opcoes: ")

      # Consultar viagens disponíveis tendo em conta a opcao do cliente
      if ordenacao =='1':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado\
                       FROM viagem WHERE duracao >= %s  AND estado = 'por realizar' ORDER BY duracao ASC", (duracao))
          viagens = cur.fetchall()
      elif ordenacao == '2':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE duracao >= %s  AND estado = 'por realizar'  ORDER BY data_de_partida ASC", (duracao))
          viagens = cur.fetchall()
      elif ordenacao == '3':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE duracao >= %s AND estado = 'por realizar' ORDER BY distancia ASC", (duracao))
          viagens = cur.fetchall()
      elif ordenacao == '4':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE duracao >= %s AND estado = 'por realizar' ", (duracao))
          viagens = cur.fetchall()
          #Se o utilizador inserir um caracter invalido assumimos que nao quer ordenar as suas viagens
      else:
          print("\n\t\t\tOpção inválida! As viagens nao serao ordenadas")
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE duracao >= %s AND estado = 'por realizar' ", (duracao))
          viagens = cur.fetchall()
          time.sleep(3)
    
      #se não houver viagens disponíveis 
      if not viagens:
          print("\n\t\t\tNão há viagens disponíveis com esses filtros de pesquisa.")
          print("\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
          msvcrt.getch()
          return   #retorna à função menu cliente
     
      headers = ["ID Viagem", "Partida", "Destino", "Data de Partida", "Data de Chegada", "Duração", "Lugares Disponíveis", "Distância", "Preço", "Estado"]
      table = []
      

      for atributo_viagem in viagens:
          table.append([atributo_viagem[0], atributo_viagem[1], atributo_viagem[2], atributo_viagem[3], atributo_viagem[4], atributo_viagem[5], atributo_viagem[6], atributo_viagem[7], atributo_viagem[8], atributo_viagem[9]])
          
      #Exibir a tabela tendo em conta os filtros de pesquisa do cliente         
      print("\n\n")
      print(tabulate(table, headers=headers))  
      print("\n\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
      msvcrt.getch()
      return  
  
  #FILTRAR POR DISTANCIA DE COIMBRA
  elif filtros == '3':
      
      os.system('cls')

      distancia = input("\t\t\tDistancia de Coimbra: ")

      #OPCAO PARA ORDENAR AS VIAGENS
      print("\n\t\t\tOPCOES:")
      print("\t\t\t1) Ordenar as viagens disponiveis por duracao")
      print("\t\t\t2) Ordenar as viagens disponiveis por data e hora") 
      print("\t\t\t3) Ordenar viagens disponiveis por distancia de Coimbra")
      print("\t\t\t4) Nao ordenar as viagens disponiveis")
      ordenacao = input("\t\t\tEscolha uma das opcoes: ")

      # Consultar viagens disponíveis tendo em conta a opcao do cliente
      if ordenacao =='1':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado\
                       FROM viagem WHERE distancia >= %s  AND estado = 'por realizar' ORDER BY duracao ASC", (distancia))
          viagens = cur.fetchall()
      elif ordenacao == '2':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE distancia >= %s  AND estado = 'por realizar'  ORDER BY data_de_partida ASC", (distancia))
          viagens = cur.fetchall()
      elif ordenacao == '3':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE distancia >= %s AND estado = 'por realizar' ORDER BY distancia ASC", (distancia))
          viagens = cur.fetchall()
      elif ordenacao == '4':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE distancia >= %s AND estado = 'por realizar' ", (distancia))
          viagens = cur.fetchall()
          #Se o utilizador inserir um caracter invalido assumimos que nao quer ordenar as suas viagens
      else:
          print("\n\t\t\tOpção inválida! As viagens nao serao ordenadas")
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE distancia >= %s AND estado = 'por realizar' ", (distancia))
          viagens = cur.fetchall()
          time.sleep(3)
    
      #se não houver viagens disponíveis 
      if not viagens:
          print("\n\t\t\tNão há viagens disponíveis com esses filtros de pesquisa.")
          print("\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
          msvcrt.getch()
          return   #retorna à função menu cliente
     
      headers = ["ID Viagem", "Partida", "Destino", "Data de Partida", "Data de Chegada", "Duração", "Lugares Disponíveis", "Distância", "Preço", "Estado"]
      table = []
      

      for atributo_viagem in viagens:
          table.append([atributo_viagem[0], atributo_viagem[1], atributo_viagem[2], atributo_viagem[3], atributo_viagem[4], atributo_viagem[5], atributo_viagem[6], atributo_viagem[7], atributo_viagem[8], atributo_viagem[9]])
          
      #Exibir a tabela tendo em conta os filtros de pesquisa do cliente         
      print("\n\n")
      print(tabulate(table, headers=headers))  
      print("\n\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
      msvcrt.getch()
      return  
  
  #NAO FILTRAR VIAGENS
  elif filtros == '4':
      os.system('cls')
      #OPCAO PARA ORDENAR AS VIAGENS
      print("\n\t\t\tOPCOES:")
      print("\t\t\t1) Ordenar as viagens disponiveis por duracao")
      print("\t\t\t2) Ordenar as viagens disponiveis por data e hora") 
      print("\t\t\t3) Ordenar viagens disponiveis por distancia de Coimbra")
      print("\t\t\t4) Nao ordenar as viagens disponiveis")
      ordenacao = input("\t\t\tEscolha uma das opcoes: ")

      # Consultar viagens disponíveis tendo em conta a opcao do cliente
      if ordenacao =='1':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado\
                       FROM viagem WHERE estado = 'por realizar' ORDER BY duracao ASC")
          viagens = cur.fetchall()
      elif ordenacao == '2':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE estado = 'por realizar'  ORDER BY data_de_partida ASC")
          viagens = cur.fetchall()
      elif ordenacao == '3':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE distancia AND estado = 'por realizar' ORDER BY distancia ASC")
          viagens = cur.fetchall()
      elif ordenacao == '4':
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE estado = 'por realizar'")
          viagens = cur.fetchall()
          #Se o utilizador inserir um caracter invalido assumimos que nao quer ordenar as suas viagens
      else:
          print("\n\t\t\tOpção inválida! As viagens nao serao ordenadas")
          cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, estado \
                       FROM viagem WHERE estado = 'por realizar'")
          viagens = cur.fetchall()
          time.sleep(3)
    
      #se não houver viagens disponíveis 
      if not viagens:
          print("\n\t\t\tNão há viagens disponíveis com esses filtros de pesquisa.")
          print("\t\t\tPressione qualquer tecla para regressar ao Menu Principal...")
          msvcrt.getch()
          return   #retorna à função menu cliente
    
      #adicionar à tabela mais uma coluna com o indice da viagem que vai permitir que o utilizador, através do exibição da tabela, escolha a viagem que mais lhe agrade 
      headers = ["ID Viagem", "Partida", "Destino", "Data de Partida", "Data de Chegada", "Duração", "Lugares Disponíveis", "Distância", "Preço", "Estado"]
      table = []
      

      for atributo_viagem in viagens:
          table.append([atributo_viagem[0], atributo_viagem[1], atributo_viagem[2], atributo_viagem[3], atributo_viagem[4], atributo_viagem[5], atributo_viagem[6], atributo_viagem[7], atributo_viagem[8], atributo_viagem[9]])
          
                
      print("\n\n")
      print(tabulate(table, headers=headers))  
      print("\n\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
      msvcrt.getch()
      return 
  
  else:
     print("\n\t\t\tOpção inválida!")
     print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
     msvcrt.getch()
     return
       

####################  OPCAO 3 #####################


def gerir_reservas(nif2):
    #limpar terminal
    os.system('cls')

    # Obter todas as reservas utilizando ou nao filtros para as ordenar 

    print("\t\t\tOPCOES:")
    print("\t\t\t1) Ordenar as reservas por duracao da viagem")
    print("\t\t\t2) Ordenar as reservas por data da viagem") 
    print("\t\t\t3) Ordenar as reservas por distancia do local de destino em relacao a Coimbra")
    print("\t\t\t4) Nao ordenar as reservas disponiveis")
    ordenacao = input("\t\t\tEscolha uma das opcoes: ")

    if ordenacao == '1':
        cur.execute("SELECT reserva.id_reserva_, reserva.estado, reserva.data, reserva.valor_pago, reserva.lugar_, viagem.duracao \
             FROM reserva \
             JOIN viagem ON reserva.viagem_id_viagem_ = viagem.id_viagem_ \
             WHERE reserva.cliente_pessoa_nif = %s \
             ORDER BY viagem.duracao ASC", str(nif2))
        reservas = cur.fetchall()
        headers = ["ID Reserva", "Estado", "Data da reserva", "Valor pago", "Lugar", "Duracao da Viagem"]
        table = []
        for atributo_reserva in reservas:
            table.append([atributo_reserva[0], atributo_reserva[1], atributo_reserva[2], atributo_reserva[3], atributo_reserva[4], atributo_reserva[5]])   
        print("\n\n")
        print(tabulate(table, headers=headers))
        if not reservas:
            print("\n\t\t\tLamentamos mas não tem qualquer reserva efetuada.")
            print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
            msvcrt.getch()
            return   #retorna à função menu cliente
        
    elif ordenacao == '2':
        cur.execute("SELECT reserva.id_reserva_, reserva.estado, reserva.data, reserva.valor_pago, reserva.lugar_, viagem.data_de_partida \
             FROM reserva \
             JOIN viagem ON reserva.viagem_id_viagem_ = viagem.id_viagem_ \
             WHERE reserva.cliente_pessoa_nif = %s \
             ORDER BY viagem.data_de_partida DESC", str(nif2))
        reservas = cur.fetchall()
        headers = ["ID Reserva", "Estado", "Data da reserva", "Valor pago", "Lugar", "Data da Viagem"]
        table = []
        for atributo_reserva in reservas:
            table.append([atributo_reserva[0], atributo_reserva[1], atributo_reserva[2], atributo_reserva[3], atributo_reserva[4], atributo_reserva[5]])   
        print("\n\n")
        print(tabulate(table, headers=headers))
        #se não tiver reservas  
        if not reservas:
            print("\n\t\t\tLamentamos mas não tem qualquer reserva efetuada.")
            print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
            msvcrt.getch()
            return   #retorna à função menu cliente
        
    elif ordenacao == '3':
        cur.execute("SELECT reserva.id_reserva_, reserva.estado, reserva.data, reserva.valor_pago, reserva.lugar_, viagem.distancia \
             FROM reserva \
             JOIN viagem ON reserva.viagem_id_viagem_ = viagem.id_viagem_ \
             WHERE reserva.cliente_pessoa_nif = %s \
             ORDER BY viagem.distancia ASC", str(nif2))
        reservas = cur.fetchall()
        headers = ["ID Reserva", "Estado", "Data da reserva", "Valor pago", "Lugar", "Distancia da Viagem"]
        table = []
        for atributo_reserva in reservas:
            table.append([atributo_reserva[0], atributo_reserva[1], atributo_reserva[2], atributo_reserva[3], atributo_reserva[4], atributo_reserva[5]])   
        print("\n\n")
        print(tabulate(table, headers=headers))
        if not reservas:
            print("\n\t\t\tLamentamos mas não tem qualquer reserva efetuada.")
            print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
            msvcrt.getch()
            return   #retorna à função menu cliente
        
    elif ordenacao == '4':
        cur.execute("Select id_reserva_, estado, data, valor_pago, lugar_ FROM reserva WHERE cliente_pessoa_nif=%s", str(nif2))
        reservas=cur.fetchall()
        headers = ["ID Reserva", "Estado", "Data da reserva", "Valor pago", "Lugar"]
        table = []
        for atributo_reserva in reservas:
            table.append([atributo_reserva[0], atributo_reserva[1], atributo_reserva[2], atributo_reserva[3], atributo_reserva[4]])   
        print("\n\n")
        print(tabulate(table, headers=headers))

    #Se o utilizador inserir um caracter invalido assumimos que nao quer ordenar as viagens das suas reservas
    else:
        print("\n\t\t\tOpção inválida! Vamos assumir que nao quer ordenar as viagens das suas reservas")
        time.sleep(3)
        cur.execute("Select id_reserva_, estado, data, valor_pago, lugar_ FROM reserva WHERE cliente_pessoa_nif=%s", str(nif2))
        reservas=cur.fetchall()
        headers = ["ID Reserva", "Estado", "Data da reserva", "Valor pago", "Lugar"]
        table = []
        for atributo_reserva in reservas:
            table.append([atributo_reserva[0], atributo_reserva[1], atributo_reserva[2], atributo_reserva[3], atributo_reserva[4]])   
        print("\n\n")
        print(tabulate(table, headers=headers))


    print("\n\t\t\tOpções:")
    print("\t\t\t1) Cancelar uma reserva")
    print("\t\t\t2) voltar ao menu do cliente")
    
    opcao = input("\n\t\t\tEscolha uma opção: ")

    # Executar ação escolhida
    if opcao == "1":
        # chama funcao para cancelar reserva
        cancelar_reserva(nif2)
        return

    elif opcao  == "2":
        return
    
    else:
        print("\n\t\t\tOpção inválida!")
        print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
        msvcrt.getch()
        return


def cancelar_reserva(nif3):
    #limpar terminal
    os.system('cls')

    #METER GOLD OU NORMAL PARA OS CLIENTES 
    #(-----)

    while True:
        # Obter todas as reservas que possam ser canceladas
        cur.execute("SELECT id_reserva_, estado, data, valor_pago, lugar_, viagem_id_viagem_ FROM reserva WHERE estado IN ('ativa', 'pendente') AND cliente_pessoa_nif = %s", nif3)
        reservas = cur.fetchall()

        #Obter estatuto do cliente em questão
        cur.execute("SELECT tipo_gold from cliente WHERE nif=%s", nif3)
        estatuto=cur.fetchone()["tipo_gold"]

        #se não tiver reservas  
        if not reservas:
            print("\n\t\t\tLamentamos mas não tem qualquer reserva ativa ou pendente.")
            print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
            msvcrt.getch()
            return   #retorna à função menu cliente
    
        #Caso contrário exibe reservas
        headers = ["ID Reserva", "Estado", "Data da reserva", "Valor pago", "Lugar", "ID Viagem", "Indice da Reserva"]
        indice=1
        table = []
        
        for atributo_reserva in reservas:
            table.append([atributo_reserva[0], atributo_reserva[1], atributo_reserva[2], atributo_reserva[3], atributo_reserva[4], atributo_reserva[5], indice])
            indice += 1
        
        print("\t\t\t--------RESERVAS PARA VIAGENS POR REALIZAR---------")
        print(tabulate(table, headers=headers))

        opcao = input("\n\t\t\tPretende cancelar alguma?(S/N): ")

        #enviar notificação se alguem tiver uma reserva pendente com o id viagem desta viagem cancelada

        if (opcao == 'S' or opcao == 's'):
            
            escolha = input("\t\t\tDigite o Índice da reserva que deseja cancelar: ")
    
            if escolha.isdigit() and int(escolha) in range(1, len(reservas)+1):
            
                #Seleciona e cancela reserva
                reserva_cancelar = reservas[int(escolha)-1]
                cur.execute("UPDATE reserva SET estado = 'Cancelada' WHERE cliente_pessoa_nif = %s AND id_reserva_=%s", (nif3, reserva_cancelar[0]))
                conn.commit()

                #se o cliente for do tipo "normal" é reembolsado em 50% e o atributo "valor pago" atualiza para 50% do seu valor 
                if (estatuto==False):
                    cur.execute("UPDATE reserva SET valor_pago = %s WHERE cliente_pessoa_nif=%s AND id_reserva_=%s", (0.50*reserva_cancelar[3],nif3, reserva_cancelar[0]))
                    conn.commit()
                

                #ISTO DEVERIA SER IMPLEMENADO ATE 2 DIAS ANTES DA VIAGEM ???? PARA TIPO GOLD E 1 SEMANA PARA TIPO NORMAL

                #Se o cliente for do tipo Gold eh reembolsado na totalidade
                elif(estatuto==True):
                    cur.execute("UPDATE reserva SET valor_pago = 0 WHERE cliente_pessoa_nif=%s AND id_reserva_=%s", (nif3, reserva_cancelar[0]))
                    conn.commit()
                print("\n\t\t\tA sua reserva foi cancelada!")

                #Aumenta um nos lugares disponiveis dessa viagem
                cur.execute("SELECT lugares_disponiveis FROM viagem WHERE id_viagem_ = %s", (reserva_cancelar[5],))
                lugares_disponiveis = cur.fetchone()["lugares_disponiveis"] 
                novo_lugares_disponiveis = lugares_disponiveis + 1
                cur.execute("UPDATE viagem SET lugares_disponiveis = %s WHERE id_viagem_ = %s", (novo_lugares_disponiveis, reserva_cancelar[5]))
                conn.commit()

                print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
                msvcrt.getch()
                return
                
            else:
                print("\n\t\t\tOpção inválida!")
                print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente...")
                msvcrt.getch()
                return 
                
        elif (opcao == 'N' or opcao =='n'):
        
            print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente")
            msvcrt.getch()
            return
        else:
            print("\n\t\t\tOpção inválida!")
            print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente")
            msvcrt.getch()
            return 


####################  OPCAO 4 #####################

def visualizar_notificacoes(nif):
    #Limpar terminal
    os.system('cls')

    #Obter as mensagens para o cliente em questão 
    cur.execute("SELECT mensagem.id_mensagem, mensagem.data_de_envio, mensagem_lida.lida \
    FROM mensagem, mensagem_lida \
    WHERE mensagem_lida.mensagem_id_mensagem = mensagem.id_mensagem AND mensagem_lida.cliente_pessoa_nif = %s \
    ORDER BY mensagem_lida.lida DESC, mensagem.data_de_envio DESC", (nif,))
    mensagens = cur.fetchall()

    if not mensagens:
        print("\n\n\t\t\tAinda não recebeu qualquer mensagem!")
        print("\t\t\tPressione qualquer tecla para regressar ao Menu Cliente...")
        msvcrt.getch()
        return
    
    #Construcao da tabela inicial para mostrar as mensagens disponiveis 
    print("\t\t\t------- NOTIFICAÇÕES E MENSAGENS ----------\n\n")
    headers = ["ID Mensagem", "Data de envio", "Lida",  "Tipo de mensagem", "Indice da Mensagem"]
    indice=1
    table = []
        
    #Percorrer a lista obtida    
    for mensagem in mensagens:

        #Isto vai-nos permitir verificar se a mensagem é automatica ou se foi enviada por um admin
        cur.execute("SELECT administrador_pessoa_nif FROM mensagem_administrador WHERE mensagem_id_mensagem = %s", (mensagem[0],))
        resultado = cur.fetchone()

        if resultado is not None: #Foi enviada por um administrador

            #obter o nome do admin
            cur.execute("SELECT nome_completo FROM administrador WHERE nif = %s", (resultado[0],))
            nome = cur.fetchone()[0]

            tipo = "Enviada por " + nome

        else: #Foi enviada por um processo automatico
            tipo = "Automática"
            

        table.append([mensagem[0], mensagem[1], mensagem[2], tipo, indice])
        indice += 1
        
    print(tabulate(table, headers=headers))

    #O utilizador vai selecionar a mensagem que pretende ler
    #O conteudo so e exibido depois do cliente selecionar a mensagem que pretende ler, como acontece nos telemoveis por exemplo
    escolha=input("\n\n\t\t\tDigite o indice da mensagem que pretende ler: ")

    if escolha.isdigit() and int(escolha) in range(1, len(mensagens)+1):
        #Limpar terminal
        os.system('cls')
        print("\t\t\t------- NOTIFICAÇÕES E MENSAGENS ----------\n\n")
            
        #Escolha da mensagem    
        mensagem = mensagens[int(escolha)-1]

        #Obter conteudo da mensagem
        cur.execute("SELECT conteudo FROM mensagem WHERE id_mensagem = %s", (mensagem[0],))
        conteudo=cur.fetchone()['conteudo']

        #Exibir mensagem
        print("\t\t\tMENSAGEM: " + conteudo)

        #Indicar mensagem como lida
        cur.execute("UPDATE mensagem_lida SET lida = 'True' WHERE mensagem_id_mensagem = %s", (mensagem[0],))
        conn.commit()

        #Regressar
        print("\n\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente")
        msvcrt.getch()
        return 
    
    else:
        print("\n\t\t\tOpção inválida!")
        print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu Cliente")
        msvcrt.getch()
        return 


        
###################################################
##########    FUNÇÕES AUXILIARES       ############
###################################################
###################################################

def mostra_viagens(estado):
    #Esta funçao exibe todas as viagens da base de dados com o estado passado como argumento da função. Não mostra
    #todos os detalhes de cada viagem, apenas os mais importantes

    # Consultar a base de dados e obter todas as viagens com o estado
    cur.execute("SELECT id_viagem_, partida, destino, data_de_partida, duracao, lugares_disponiveis, preco_base_ FROM viagem WHERE estado = %s", (estado,))
    viagens = cur.fetchall()

    # Verificar se há viagens a apresentar
    if not viagens:
        return 0

    #Exibicao dos autocarros disponíveis numa tabela 
    headers = ["ID Viagem", "Partida", "Destino", "Data de partida", "Duração (min)", "Lugares disponíveis", "Preço (euros)"]
    table = []

    for viagem in viagens:
        table.append([viagem[0], viagem[1], viagem[2], viagem[3], viagem[4], viagem[5], viagem[6]])

    # Apresentar as viagens numa tabela utilizando a biblioteca "tabulate". 
    print("------ VIAGENS DO TIPO '", estado,"' -----\t\t\t")
    print(tabulate(table, headers=headers, tablefmt="grid"))

    return 1


def mostra_viagem(ID):
    #Esta funçao exibe os detalhes da viagem cujo ID é passado como argumento

    # Consultar a base de dados e obter a viagem
    cur.execute("SELECT partida, destino, data_de_partida, data_de_chegada, duracao, lugares_disponiveis, preco_base_, distancia, autocarro_matricula, estado FROM viagem WHERE id_viagem_ = %s", (ID,))
    viagem = cur.fetchall()

    # Verificar se a viagem existe
    if not viagem:
        return(0)
    else:
        viagem = viagem[0]

    # Apresentar a viagem
    os.system('cls')
    print("------ DETALHES DA VIAGEM ID:", ID," -----\t\t\t")
    headers = ["Partida", "Destino", "Data de partida", "Data de chegada", "Duração (min)", "Lugares disponíveis", "Preço (euros)", "Distância (km)", "Autocarro", "Estado"]
    table = []
    table.append([viagem[0], viagem[1], viagem[2], viagem[3], viagem[4], viagem[5], viagem[6], viagem[7], viagem[8], viagem[9]])
    print(tabulate(table, headers=headers, tablefmt="grid"))


    #Apresentar também o respetivo histórico de alterações, caso exista
    cur.execute("SELECT id_alteracao, data, preco_antigo FROM alteracao_viagem WHERE viagem_id_viagem_ = %s ORDER BY data DESC", (ID,))
    alteracoes = cur.fetchall()

    # Verificar se existem alterações para a viagem selecionada
    if alteracoes:
        # Apresentar as alterações numa tabela formatada
        print("\n\n\n\t\t\t ----- HISTÓRICO DE ALTERAÇÕES ------\n")
        table = []

        for alteracao in alteracoes:
            table.append([alteracao[0], alteracao[1], alteracao[2]])

        print(tabulate(table, headers=['ID da Alteração', 'Data da alteração', 'Preço Antigo (euros)'], tablefmt='psql'))

    print("\n\n")
    return 1


def message2client(clientes, admin):
    #Limpar o terminal
    os.system('cls')

    #Exibicao dos clientes registados numa tabela 
    headers = ["Nome", "NIF", "Telemovel", "Email", "Estatuto Gold" , "Indice do Cliente"]
    table = []
    indice = 1

    for atributo_cliente in clientes:
        table.append([atributo_cliente[0], atributo_cliente[1], atributo_cliente[2], atributo_cliente[3], atributo_cliente[4], indice])
        indice += 1
        
    print("\t\t\t-------LISTA DE CLIENTES COIMBRA BUS --------\n")
    print(tabulate(table, headers=headers))

    escolha=input("\n\t\t\tSelecione o índice do cliente ao qual pretende enviar mensagem: ")

    if escolha.isdigit() and int(escolha) in range(1, len(clientes)+1):
            #Determinar cliente escolhido
            cliente_escolhido = clientes[int(escolha)-1]  

    else:
        print("\n\t\t\tOpção inválida!")
        print("\t\t\tPressione qualquer tecla para ser redirecionado para o Menu de Envio de Mensagens...")
        msvcrt.getch()
        return


    while True:
        #Indicar o destinatario e pedir para escrever mensagem
        os.system('cls')
        print(f"\t\t\t------- ENVIAR MENSAGEM PARA {cliente_escolhido[0]} --------")
        mensagem = input("\n\t\t\tESCREVA A MENSAGEM PRETENDIDA (deve ter pelo menos 5 caracteres): ")       
        
        if len(mensagem) >= 5:
            break
        else:
            print("\n\n\t\t\tA mensagem deve ter pelo menos 5 caracteres.")
            print("\n\n\t\t\tTente outra vez...")
            time.sleep(2)
            continue

    # 1 - Insere a mensagem na tabela de mensagens
    cur.execute("INSERT INTO mensagem (data_de_envio, conteudo) VALUES (DATE_TRUNC('minute', CURRENT_TIMESTAMP), %s)", (mensagem,))
    conn.commit()

    # 2- Inserir na tabela 'mensagem_lida' a entrada correspondente (permite depois saber se o cliente em questão já leu a mensagem)
    #Obtém o id da última mensagem inserida
    cur.execute("SELECT MAX(id_mensagem) FROM mensagem")
    id_mensagem = cur.fetchone()[0]

    # Insere uma nova entrada na tabela 'mensagem_lida'
    lida = False  # Mensagem ainda não lida
    cur.execute("INSERT INTO mensagem_lida (lida, mensagem_id_mensagem, cliente_pessoa_nif) VALUES (%s, %s, %s)", (lida, id_mensagem, cliente_escolhido[1]))
    conn.commit()

    # 3 - Inserir na tabela 'mensagem_administrador' a entrada respetiva
    # Esta tabela permite-nos associar uma determinada mensagem a um determinado utilizador
    nif = admin['nif']
    cur.execute("INSERT INTO mensagem_administrador (mensagem_id_mensagem, administrador_pessoa_nif) VALUES (%s, %s)", (id_mensagem, nif))
    conn.commit()

    print("\n\n\t\t\tMensagem enviada com sucesso!")
    print("\t\t\tPressione qualquer tecla para regressar ao Menu de Envio de Mensagens...")
    msvcrt.getch()
    return


def message2all(clientes, admin):
    #Limpar o terminal
    os.system('cls')

    while True:
        #Indicar o destinatario e pedir para escrever mensagem
        os.system('cls')
        print(f"\t\t\t------- ENVIAR MENSAGEM PARA TODOS OS CLIENTES --------")
        mensagem = input("\n\t\t\tESCREVA A MENSAGEM PRETENDIDA (deve ter pelo menos 5 caracteres): ")       
        
        if len(mensagem) >= 5:
            break
        else:
            print("\n\n\t\t\tA mensagem deve ter pelo menos 5 caracteres.")
            print("\n\n\t\t\tTente outra vez...")
            time.sleep(2)
            continue

    # 1 - Insere a mensagem na tabela de mensagens
    cur.execute("INSERT INTO mensagem (data_de_envio, conteudo) VALUES (DATE_TRUNC('minute', CURRENT_TIMESTAMP), %s)", (mensagem,))
    conn.commit()

    # 2- Inserir na tabela 'mensagem_lida' a entrada correspondente (permite depois saber se o cliente em questão já leu a mensagem)
    #Obtém o id da última mensagem inserida
    cur.execute("SELECT MAX(id_mensagem) FROM mensagem")
    id_mensagem = cur.fetchone()[0]

    # Insere uma nova entrada na tabela 'mensagem_lida' para cada cliente
    lida = False  # Mensagem ainda não lida

    for cliente in clientes:
        nif = cliente['nif']
        cur.execute("INSERT INTO mensagem_lida (lida, mensagem_id_mensagem, cliente_pessoa_nif) VALUES (%s, %s, %s)", (lida, id_mensagem, nif))
        conn.commit()


    # 3 - Inserir na tabela 'mensagem_administrador' a entrada respetiva
    # Esta tabela permite-nos associar uma determinada mensagem a um determinado utilizador
    nif = admin['nif']
    cur.execute("INSERT INTO mensagem_administrador (mensagem_id_mensagem, administrador_pessoa_nif) VALUES (%s, %s)", (id_mensagem, nif))
    conn.commit()

    print("\n\n\t\t\tMensagem enviada com sucesso!")
    print("\t\t\tPressione qualquer tecla para regressar ao Menu de Envio de Mensagens...")
    msvcrt.getch()
    return

def lugar_associado(ID1):

    cur.execute("SELECT lugares_disponiveis FROM viagem WHERE id_viagem_ = %s", (ID1,))
    lugares_disponiveis = cur.fetchone()[0]

    # Obter a lista de lugares reservados para a viagem, ordenados pelo número do lugar
    cur.execute("SELECT lugar_ FROM reserva WHERE viagem_id_viagem_ = %s ORDER BY lugar_", (ID1,))
    lugares_reservados = [r[0] for r in cur.fetchall()]

    # Encontrar o primeiro lugar disponível
    for i in range(1, lugares_disponiveis + 1):
        if i not in lugares_reservados:
            lugar = i
            break

    return lugar


###################################################
##########         MAIN             ###############
###################################################
###################################################

# Estabelecer ligação 
conn = psycopg2.connect("host=localhost dbname=CoimbraBus user=postgres password=Bsoares02")

# Cria um objecto (cursor) que permite executar operações sobre a base de dados
cur = conn.cursor(cursor_factory=psycopg2.extras.DictCursor)

while True:
    # Limpa o terminal
    os.system('cls')

    print("\t\t\tBEM-VINDO AO COIMBRA BUS!\n\n")
    print("\t\t\tMENU PRINCIPAL")
    print("\t\t\t1) Login")
    print("\t\t\t2) Registar pela primeira vez")
    print("\t\t\t3) Entrar como administrador")
    print("\t\t\t4) Sair\n")

    opcao = input("\t\t\tSelecione uma opção: ")

    if opcao == "1":
        login()
        # Chamar função para fazer login com as credenciais inseridas

    elif opcao == "2":
        registo()
        # Chamar função para registar o utilizador com as informações inseridas

    elif opcao == "3":
        admin_login()
        # Chamar função para fazer login de administrador com as credenciais inseridas

    elif opcao == "4":
        os.system('cls')

        # Fecha a ligação à base de dados
        cur.close()
        conn.close()

        #Indicar término do programa
        print("\n\n\t\t\tPROGRAMA TERMINADO")
        print("\t\t\tA JANELA IRÁ FECHAR BREVEMENTE...")
        time.sleep(2)
        os.system('cls')
        break

    else:
            print("\n\n\t\t\tOpção inválida. Tente novamente.")
            print("\t\t\tPressione qualquer tecla para continuar...")
            msvcrt.getch()
