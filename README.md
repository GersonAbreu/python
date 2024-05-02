import tkinter as tk
from tkinter import ttk, filedialog, messagebox
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

# Dicionário com informações de cada produto: produto -> (marca, validade, valor)
produtos_info = {
    "Pão de Forma": ("Panco", "15/03/2025", "R$10,99"),
    "Requeijão": ("Vigor", "17/02/2026", "R$7,80"),
    "Arroz": ("Camil", "19/06/2026", "R$30,00"),
    "Macarrão": ("Adria", "18/04/2024", "R$4,80"),
    "Leite": ("Mococa", "25/12/2026", "R$4,50"),
    "Farinha de Trigo": ("Dona Benta", "30/01/2026", "R$3,80"),
    "Extrato de Tomate": ("Pomarola", "25/11/2026", "R$2,90")
}

def popup(message):
    messagebox.showinfo("Pop-up", message)

def criar_nova_compra():
    produtos_selecionados.clear()
    notebook.select(frame_nome)

def atualizar_resumo():
    nome = nome_var.get()
    endereco = endereco_var.get()
    
    texto_resumo = f"Nome: {nome}\nEndereço: {endereco}\n\n"  
    
    for produto in produtos_selecionados:
        texto_resumo += f"Produto: {produto}\n"
        texto_resumo += f"Marca: {produtos_info[produto][0]}\n"
        texto_resumo += f"Validade: {produtos_info[produto][1]}\n"
        texto_resumo += f"Valor: {produtos_info[produto][2]}\n"
        texto_resumo += "-" * 20 + "\n"
    
    valor_total = sum([float(produtos_info[produto][2].replace("R$", "").replace(",", ".")) for produto in produtos_selecionados])
    texto_resumo += f"\nValor Total: R${valor_total:.2f}"
    
    resumo.delete(1.0, tk.END)
    resumo.insert(tk.END, texto_resumo)

def atualizar_resumo_nome():
    nome = nome_var.get()
    if nome:
        popup(f"Nome '{nome}' enviado com sucesso!")
        atualizar_resumo()

def atualizar_resumo_endereco():
    endereco = endereco_var.get()
    if endereco:
        popup(f"Endereço '{endereco}' enviado com sucesso!")
        atualizar_resumo()

def adicionar_produto():
    produto_selecionado = comboProdutos.get().split(' - ')[0]  # Obtém apenas o nome do produto
    produtos_selecionados.append(produto_selecionado)
    popup(f"Produto '{produto_selecionado}' adicionado com sucesso!")
    atualizar_resumo()  # Chamando a função para atualizar o resumo após adicionar o produto

def remover_produto():
    if produtos_selecionados:
        produto_removido = produtos_selecionados.pop()
        popup(f"Produto '{produto_removido}' removido com sucesso!")
        atualizar_resumo()

def salvar_compra_em_pdf():
    resumo_texto = resumo.get(1.0, tk.END).strip()
    file_path = filedialog.asksaveasfilename(defaultextension=".pdf", filetypes=[("PDF files", "*.pdf")])
    if file_path:
        c = canvas.Canvas(file_path, pagesize=letter)
        
        y = 750  # Posição inicial na página
        
        linhas = resumo_texto.split('\n')
        
        for linha in linhas:
            c.drawString(50, y, linha)
            y -= 15
            
        c.save()
        
        popup(f"Resumo da compra salvo como '{file_path}'.")

produtos_selecionados = []

janela = tk.Tk()
janela.title("Pão de Queijo e Mortadela")
janela.geometry("800x600")
janela.maxsize(width=1200, height=850)
janela.minsize(width=500, height=300)
janela.configure(background="purple")

nome_var = tk.StringVar()
endereco_var = tk.StringVar()

notebook = ttk.Notebook(janela)
notebook.pack(fill=tk.BOTH, expand=True)

frame_nome = ttk.Frame(notebook)
notebook.add(frame_nome, text="Nome")
ttk.Label(frame_nome, text="Digite seu nome completo:").pack()
entrada_nome = ttk.Entry(frame_nome, textvariable=nome_var)
entrada_nome.pack()
ttk.Button(frame_nome, text="Enviar Nome", command=atualizar_resumo_nome).pack()

frame_endereco = ttk.Frame(notebook)
notebook.add(frame_endereco, text="Endereço")
ttk.Label(frame_endereco, text="Digite seu endereço completo:").pack()
entrada_endereco = ttk.Entry(frame_endereco, textvariable=endereco_var)
entrada_endereco.pack()
ttk.Button(frame_endereco, text="Enviar Endereço", command=atualizar_resumo_endereco).pack()

frame_produtos = ttk.Frame(notebook)
notebook.add(frame_produtos, text="Produtos")
ttk.Label(frame_produtos, text="Produtos:").pack()
# Modificando a forma como os itens são exibidos no combobox para incluir o valor do produto
produtos_combobox = [f"{produto} - {info[2]}" for produto, info in produtos_info.items()]
comboProdutos = ttk.Combobox(frame_produtos, values=produtos_combobox)
comboProdutos.pack()
ttk.Button(frame_produtos, text="Adicionar Produto", command=adicionar_produto).pack()

frame_resumo = ttk.Frame(notebook)
notebook.add(frame_resumo, text="Resumo")
resumo_scrollbar = ttk.Scrollbar(frame_resumo)
resumo_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
resumo = tk.Text(frame_resumo, width=60, height=20)
resumo.pack(pady=10)
resumo.config(yscrollcommand=resumo_scrollbar.set)
resumo_scrollbar.config(command=resumo.yview)


ttk.Button(frame_resumo, text="Atualizar Resumo", command=atualizar_resumo).pack(pady=5)
ttk.Button(frame_resumo, text="Remover Último Produto", command=remover_produto).pack(pady=5)
ttk.Button(frame_resumo, text="Nova Compra", command=criar_nova_compra).pack()
ttk.Button(janela, text="Finalizar Compra", command=salvar_compra_em_pdf).pack(pady=10)

janela.mainloop()
