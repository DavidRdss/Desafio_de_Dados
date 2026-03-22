# Desafios que fiz só

# Desafio01.py

def manutencao_mensal_bug(contas):
    for i in range(len(contas)):
        if contas[i]['saldo'] < 0:
            print(f"Removendo (bug) da conta id={contas[i]['id']} saldo={contas[i]['saldo']}")
            contas.pop(i)

def manutencao_mensal_fix(contas):
    antes = len(contas)
    contas[:] = [c for c in contas if c['saldo'] >= 0]
    print(f"Removidas (fix): {antes - len(contas)} contas com saldo negativo.")

def manutencao_mensal_fix_inplace_reverse(contas):
    for i in range(len(contas) - 1, -1, -1):
        if contas[i]['saldo'] < 0:
            print(f"Removendo (reverse) conta id={contas[i]['id']} saldo={contas[i]['saldo']}")
            contas.pop(i)

def cria_contas_exemplo():
    return [
        {'id': 1, 'titular': 'A', 'saldo': 100.0},
        {'id': 2, 'titular': 'B', 'saldo': -20.0},
        {'id': 3, 'titular': 'C', 'saldo': -5.0},
        {'id': 4, 'titular': 'D', 'saldo': 50.0},
        {'id': 5, 'titular': 'E', 'saldo': -1.0},
    ]

if __name__ == "__main__":
    print("=== Demonstração do BUG (Índice Fantasma) ===")
    contas = cria_contas_exemplo()
    print("Antes:", contas)
    try:
        manutencao_mensal_bug(contas)
        print("Depois (bug):", contas)
    except Exception as e:
        print("Erro reproduzido pela versão bugada:", repr(e))

  print("\n=== Correção (comprehension in-place) ===")
    contas2 = cria_contas_exemplo()
    print("Antes:", contas2)
    manutencao_mensal_fix(contas2)
    print("Depois (fix):", contas2)

  print("\n=== Correção alternativa (iterar inverso) ===")
    contas3 = cria_contas_exemplo()
    print("Antes:", contas3)
    manutencao_mensal_fix_inplace_reverse(contas3)
    print("Depois (reverse fix):", contas3)

  # Desafio02.py

from dataclasses import dataclass, field
from datetime import date
import uuid

@dataclass
class EquipamentoTI:
    id_patrimonio: str = field(default_factory=lambda: str(uuid.uuid4()))
    tipo: str = ""
    marca: str = ""
    valor_compra: float = 0.0
    esta_ativo: bool = True
    data_aquisicao: date = field(default_factory=date.today)

  def __post_init__(self):
        self.valor_compra = float(self.valor_compra)

  def __str__(self):
        status = "ATIVO" if self.esta_ativo else "INATIVO"
        return f"[{status}] {self.tipo} {self.marca} (Patrimônio: {self.id_patrimonio}) - R${self.valor_compra:.2f}"

  def __repr__(self):
        return (
            f"EquipamentoTI(id={self.id_patrimonio!r}, tipo={self.tipo!r}, "
            f"marca={self.marca!r}, valor={self.valor_compra:.2f}, ativo={self.esta_ativo})"
        )

if __name__ == "__main__":
    e1 = EquipamentoTI(tipo="Notebook", marca="Dell", valor_compra=4500.00)
    e2 = EquipamentoTI(tipo="Monitor", marca="LG", valor_compra=900.50, esta_ativo=False)
    print(e1)
    print(repr(e1))
    print(e2)

  # Desafio03.py

from dataclasses import dataclass
from typing import Optional

@dataclass
class SensorAmbiente:
    localizacao: str
    temperatura: Optional[float] = None
    umidade: Optional[float] = None

  def atualizar_leitura(self, nova_temp: float, nova_umidade: float):
        erros = []
        if not (-50.0 <= nova_temp <= 100.0):
            erros.append(f"Temperatura {nova_temp} fora do intervalo [-50.0, 100.0].")
        if not (0.0 <= nova_umidade <= 100.0):
            erros.append(f"Umidade {nova_umidade} fora do intervalo [0, 100].")
        if erros:
            print(
                f"[SensorAmbiente:{self.localizacao}] Leitura inválida: "
                f"{' '.join(erros)} Ignorando atualização."
            )
            return

  self.temperatura = float(nova_temp)
  self.umidade = float(nova_umidade)
   print(
            f"[SensorAmbiente:{self.localizacao}] Leitura atualizada: "
            f"temp={self.temperatura}, umidade={self.umidade}"
        )

  def __str__(self):
        return f"SensorAmbiente(local={self.localizacao}, temp={self.temperatura}, umid={self.umidade})"


if __name__ == "__main__":
    s = SensorAmbiente(localizacao="Sala 101")
    s.atualizar_leitura(22.5, 45.0)
    s.atualizar_leitura(-100.0, 50.0)
    s.atualizar_leitura(25.0, 150.0)
    print(s)

  # Desafio04.py

from dataclasses import dataclass

class TransferError(Exception):
    pass

@dataclass
class ContaCorrente:
    titular: str
    saldo: float = 0.0

  def sacar(self, valor: float):
        if valor <= 0:
            raise ValueError("Valor de saque deve ser maior que 0.")
        if valor > self.saldo:
            raise ValueError("Saldo insuficiente.")
        self.saldo -= float(valor)
        return True

  def depositar(self, valor: float):
        if valor <= 0:
            raise ValueError("Valor de depósito deve ser maior que 0.")
        self.saldo += float(valor)
        return True

  def transferir(self, destino: "ContaCorrente", valor: float):
        if not isinstance(destino, ContaCorrente):
            raise TypeError("Destino deve ser uma ContaCorrente.")
        if valor <= 0:
            raise ValueError("Valor de transferência deve ser maior que 0.")
        if destino is self:
            raise TransferError("Não é permitido transferir para a mesma conta.")

  try:
    self.sacar(valor)
    destino.depositar(valor)
        except Exception as e:
            self.depositar(valor)  # rollback
            raise TransferError(f"Falha ao creditar destino: rollback efetuado. Causa: {e}")

if __name__ == "__main__":
    a = ContaCorrente("Alice", 100.0)
    b = ContaCorrente("Bruno", 20.0)

  print("Saldos iniciais:", a.saldo, b.saldo)

  try:
        a.transferir(b, 30.0)
        print("Transferência bem-sucedida.")
    except Exception as e:
        print("Transfer failed:", e)

  print("Saldos após 1ª:", a.saldo, b.saldo)

  try:
        b.transferir(a, 1000.0)
    except Exception as e:
        print("Erro esperado (saldo insuficiente):", e)

  try:
        a.transferir(b, -10.0)
    except Exception as e:
        print("Erro esperado (valor inválido):", e)

# Desafio05.py

from typing import List
from desafio02_equipamento import EquipamentoTI

class SistemaInventario:
    def __init__(self):
        self.itens = {}

  def adicionar(self, equipamento: EquipamentoTI):
        self.itens[equipamento.id_patrimonio] = equipamento

  def remover_por_id(self, id_patrimonio: str):
        self.itens.pop(id_patrimonio, None)

  def total_investido(self):
        return sum(e.valor_compra for e in self.itens.values() if e.esta_ativo)

  def filtrar_por_marca(self, marca: str) -> List[EquipamentoTI]:
        marca = marca.lower()
        return [e for e in self.itens.values() if e.marca.lower() == marca]

  def __repr__(self):
        itens = len(self.itens)
        ativos = sum(e.esta_ativo for e in self.itens.values())
        total = self.total_investido()
        return f"<SistemaInventario itens={itens} ativos={ativos} total_valor=R${total:.2f}>"

if __name__ == "__main__":
    exemplos = [
        EquipamentoTI("Notebook","Dell",4200),
        EquipamentoTI("Notebook","Dell",4800),
        EquipamentoTI("Monitor","LG",800),
        EquipamentoTI("Servidor","HP",12000,True),
        EquipamentoTI("Teclado","Logitech",120),
        EquipamentoTI("Mouse","Logitech",80),
        EquipamentoTI("Switch","Cisco",2200),
        EquipamentoTI("No-Break","APC",1500,False),
        EquipamentoTI("Notebook","Lenovo",3500),
        EquipamentoTI("Monitor","Dell",950),
    ]

  sistema = SistemaInventario()
    for e in exemplos:
        sistema.adicionar(e)

  print(repr(sistema))
  print("Total investido (ativos): R${:.2f}".format(sistema.total_investido()))

  dells = sistema.filtrar_por_marca("Dell")
  print(f"Encontrados {len(dells)} equipamentos da marca 'Dell':")
  for d in dells:
        print("  -", d)

# Explicando o desafio:

Esses códigos fazem coisas diferentes, mas todos têm a mesma ideia: organizar dados e testar regras em Python.

No desafio01, o código faz uma lista de contas bancárias. Ele tenta apagar as contas que estão com saldo negativo. A primeira função tem um erro pq remove itens da lista enquanto está passando por ela, o que pode bagunçar as posições e fazer o programa falhar ou pular alguma conta. As outras duas funções fazem isso do jeito certo, sem causar esse problema : )

No desafio02, o código cria uma classe para representar um equipamento de TI, como um notebook, monitor ou servidores da área. Ele guarda informações sendo elas os tipos, marcas, valores, se o equipamento está ativo e um número de patrimônio único. Também mostra o equipamento de forma organizada quando imprime o codigo na tela.

No desafio03, o código representa um sensor de ambiente. Esse sensor guarda a localização, a temperatura e a umidade. Quando alguém tenta atualizar os valores, o código olha se eles estão dentro de limites aceitáveis. Se tiver errados ele não atualiza nada e avisa o problema.

No desafio04, o código cria uma conta corrente com funções para sacar, depositar e transferir dinheiro. Ele também verifica se os valores são válidos. Se algo der errado na transferência, ele desfaz a operação para não deixar o saldo errado.

No desafio05, o código monta um sistema de inventário para guardar vários equipamentos de TI parecido com o desafio02. Ele permite adicionar equipamentos, remover pelo ID, somar o valor dos itens ativos e procurar equipamentos por uma marca. ele tbm mostra um resumo do inventário e lista os equipamentos de uma marca específica.
