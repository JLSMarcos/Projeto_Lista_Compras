#LISTA DE COMPRAS
import csv
import os

# Função para carregar a lista de compras do arquivo CSV
def carregar_lista():
    if not os.path.exists('lista_compras.csv'):
        return []
    
    with open('lista_compras.csv', mode='r', newline='', encoding='utf-8') as file:
        reader = csv.reader(file)
        lista = [linha for linha in reader]
    return lista

# Função para salvar a lista de compras no arquivo CSV
def salvar_lista(lista):
    with open('lista_compras.csv', mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerows(lista)

# Função para exibir a lista de compras
def exibir_lista(lista):
    if len(lista) == 0:
        print("Lista de compras vazia.")
    else:
        print("\nLista de Compras:")
        for idx, item in enumerate(lista, 1):
            print(f"{idx}. Produto: {item[0]} | Preço: R$ {item[1]} | Local: {item[2]}")

# Função para adicionar um produto à lista
def adicionar_produto(lista):
    produto = input("Digite o nome do produto: ")
    preco = input("Digite o preço do produto: ")
    local = input("Digite o local de compra: ")

    # Adicionando o produto à lista
    lista.append([produto, preco, local])
    print(f"Produto '{produto}' adicionado com sucesso!")
    salvar_lista(lista)

# Função para remover um produto da lista
def remover_produto(lista):
    exibir_lista(lista)
    
    try:
        index = int(input("\nDigite o número do produto que deseja remover: ")) - 1
        if 0 <= index < len(lista):
            item_removido = lista.pop(index)
            print(f"Produto '{item_removido[0]}' removido com sucesso!")
            salvar_lista(lista)
        else:
            print("Número inválido. Tente novamente.")
    except ValueError:
        print("Por favor, digite um número válido.")

# Função principal
def main():
    lista = carregar_lista()
    
    while True:
        print("\n--- Menu ---")
        print("1. Adicionar produto")
        print("2. Exibir lista de compras")
        print("3. Remover produto")
        print("4. Sair")
        
        escolha = input("Escolha uma opção (1-4): ")
        
        if escolha == '1':
            adicionar_produto(lista)
        elif escolha == '2':
            exibir_lista(lista)
        elif escolha == '3':
            remover_produto(lista)
        elif escolha == '4':
            print("Saindo...")
            break
        else:
            print("Opção inválida. Tente novamente.")

if __name__ == "__main__":
    main()

