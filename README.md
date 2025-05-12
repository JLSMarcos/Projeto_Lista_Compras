import os  
import csv  
from collections import Counter  

CSV_FILE = 'lista_compras.csv'         # lista de compras do usuário
HIST_FILE = 'historico_compras.csv'    # histórico de itens comprados para análise
PROMO_FILE = 'promocoes.csv'           # lista opcional de promoções (produto, preço antigo, preço novo)

class DataManager:
    """Classe responsável por gerenciar I/O de dados em CSV."""
    @staticmethod
    def load_list():
        """Carrega a lista de compras do usuário do arquivo CSV, retorna lista vazia se não existir."""
        if not os.path.exists(CSV_FILE):
            return []
        with open(CSV_FILE, newline='', encoding='utf-8') as f:
            return list(csv.reader(f))  # retorna lista de [produto, preco, local]

    @staticmethod
    def save_list(lista):
        """Salva a lista de compras no arquivo CSV, sobrescrevendo conteúdo anterior."""
        with open(CSV_FILE, 'w', newline='', encoding='utf-8') as f:
            writer = csv.writer(f)
            writer.writerows(lista)

    @staticmethod
    def load_history():
        """Carrega o histórico completo de compras do arquivo CSV, retorna lista vazia se não existir."""
        if not os.path.exists(HIST_FILE):
            return []
        with open(HIST_FILE, newline='', encoding='utf-8') as f:
            return list(csv.reader(f))  # retorna lista de [produto, preco, local] comprados

    @staticmethod
    def save_history(entry):
        """Adiciona um registro de compra ao histórico, sem apagar dados anteriores."""
        with open(HIST_FILE, 'a', newline='', encoding='utf-8') as f:
            writer = csv.writer(f)
            writer.writerow(entry)  # registra [produto, preco, local]

    @staticmethod
    def load_promotions():
        """Carrega lista de promoções (produto, preço antigo, preço atual) do CSV."""
        if not os.path.exists(PROMO_FILE):
            return []
        with open(PROMO_FILE, newline='', encoding='utf-8') as f:
            return list(csv.reader(f))  # retorna lista de [produto, preco_antigo, preco_atual]

class Recommender:
    """Classe que gera recomendações com base no produto mais barato comprado e promoções."""
    def __init__(self):
        # carrega histórico e promoções
        self.history = DataManager.load_history()
        self.promos = DataManager.load_promotions()

    def cheapest_history(self):
        """Encontra o produto mais barato já comprado no histórico."""
        if not self.history:
            return None
        cheapest = None
        min_price = float('inf')
        # percorre histórico para achar menor preço
        for prod, price, _ in self.history:
            try:
                p = float(price)
                if p < min_price:
                    min_price = p
                    cheapest = prod
            except ValueError:
                continue  # ignora valores inválidos
        return cheapest

    def cheapest_promotions(self, n=5):
        """Retorna os n produtos em promoção com menor preço atual."""
        valid_promos = []
        # filtra e ordena por preço atual
        for prod, old_price, new_price in self.promos:
            try:
                p = float(new_price)
                valid_promos.append((prod, p))
            except ValueError:
                continue  # ignora dados inválidos
        # ordena crescente pelo preço atual (menor primeiro)
        valid_promos.sort(key=lambda x: x[1])
        # retorna apenas nomes dos produtos
        return [prod for prod, _ in valid_promos[:n]]

    def recommend(self, promo_n=5):
        """Combina o produto mais barato do histórico e promoções para recomendações."""
        recs = set()
        cheapest_hist = self.cheapest_history()
        if cheapest_hist:
            recs.add(cheapest_hist)  # adiciona produto histórico mais barato
        # adiciona top n promoções mais baratas
        recs.update(self.cheapest_promotions(promo_n))
        return sorted(recs)  # retorna lista ordenada alfabeticamente

class CLI:
    """Interface de linha de comando para interação com o usuário."""
    def __init__(self):
        self.lista = DataManager.load_list()  # lista atual de compras
        self.recommender = Recommender()      # inicializa recomendador

    def display_menu(self):
        """Mostra opções disponíveis no menu principal."""
        print("\n--- Menu de Lista de Compras Inteligente ---")
        print("1. Adicionar produto")
        print("2. Exibir lista de compras")
        print("3. Remover produto")
        print("4. Recomendar lista")
        print("5. Sair")

    def add_product(self):
        """Lê dados do usuário, adiciona produto à lista e histórico."""
        prod = input("Produto: ")
        preco = input("Preço (R$): ")
        local = input("Local de compra: ")
        self.lista.append([prod, preco, local])
        DataManager.save_list(self.lista)      # persiste lista
        DataManager.save_history([prod, preco, local])  # persiste histórico
        print(f"'{prod}' adicionado com sucesso!")

    def show_list(self):
        """Exibe itens atuais na lista de compras, ou mensagem se vazia."""
        if not self.lista:
            print("Lista vazia.")
            return
        print("\nSua lista atual:")
        for idx, (prod, price, place) in enumerate(self.lista, 1):
            print(f"{idx}. {prod} | R$ {price} | {place}")

    def remove_product(self):
        """Permite remover item pelo índice exibido na lista."""
        self.show_list()
        try:
            idx = int(input("Número do item para remover: ")) - 1
            if 0 <= idx < len(self.lista):
                removed = self.lista.pop(idx)
                DataManager.save_list(self.lista)
                print(f"'{removed[0]}' removido.")
            else:
                print("Índice inválido.")
        except ValueError:
            print("Entrada inválida. Use um número.")

    def recommend_list(self):
        """Calcula e exibe lista de recomendações para o usuário."""
        recommendations = self.recommender.recommend()  # obtém sugestões
        if not recommendations:
            print("Sem recomendações disponíveis no momento.")
        else:
            print("\nItens recomendados (mais barato + promoções baratas):")
            for item in recommendations:
                print(f"- {item}")

    def run(self):
        """Loop principal: exibe menu e chama funcionalidades conforme escolha."""
        while True:
            self.display_menu()
            choice = input("Escolha: ")
            if choice == '1':
                self.add_product()
            elif choice == '2':
                self.show_list()
            elif choice == '3':
                self.remove_product()
            elif choice == '4':
                self.recommend_list()
            elif choice == '5':
                print("Saindo...")
                break
            else:
                print("Opção inválida. Tente novamente.")

# Ponto de entrada do script
if __name__ == '__main__':
    CLI().run()  # inicia a interface CLI
