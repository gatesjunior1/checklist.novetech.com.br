import React, { useState } from 'react';
import { Card, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Tabs, TabsList, TabsTrigger, TabsContent } from '@/components/ui/tabs';
import { Checkbox } from '@/components/ui/checkbox';

const atendentes = ['Raquel', 'Junior', 'Rafa', 'Isis'];
const categorias = ['urgencia', 'semanal', 'mensal'];
const prazos = {
  urgencia: 2,
  semanal: 7,
  mensal: 30,
};

const ChecklistAgendamento = () => {
  const [tarefas, setTarefas] = useState({});
  const [novaTarefa, setNovaTarefa] = useState('');
  const [modoCoordenador, setModoCoordenador] = useState(false);
  const [senha, setSenha] = useState('');
  const senhaCorreta = '1234';

  const adicionarTarefa = (nome, categoria) => {
    if (!novaTarefa) return;
    const nova = { texto: novaTarefa, feito: false, dataCriacao: new Date().toISOString() };
    setTarefas(prev => ({
      ...prev,
      [nome]: {
        ...prev[nome],
        [categoria]: [...(prev[nome]?.[categoria] || []), nova],
      },
    }));
    setNovaTarefa('');
  };

  const alternarFeito = (nome, categoria, index) => {
    setTarefas(prev => {
      const atual = [...(prev[nome]?.[categoria] || [])];
      atual[index].feito = !atual[index].feito;
      return {
        ...prev,
        [nome]: {
          ...prev[nome],
          [categoria]: atual,
        },
      };
    });
  };

  const autenticarCoordenador = () => {
    if (senha === senhaCorreta) {
      setModoCoordenador(true);
      setSenha('');
    } else {
      alert('Senha incorreta!');
    }
  };

  const verificarAtraso = (tarefa, categoria) => {
    if (tarefa.feito || !tarefa.dataCriacao) return false;
    const dias = prazos[categoria];
    const dataCriacao = new Date(tarefa.dataCriacao);
    const agora = new Date();
    const diffDias = Math.floor((agora - dataCriacao) / (1000 * 60 * 60 * 24));
    return diffDias > dias;
  };

  return (
    <div className="p-4 space-y-6">
      <div className="flex flex-col sm:flex-row sm:justify-between sm:items-center gap-4">
        <h1 className="text-2xl font-bold">Check-list de Agendamento Semanal</h1>
        {modoCoordenador ? (
          <Button onClick={() => setModoCoordenador(false)}>
            Sair do Modo Coordenador
          </Button>
        ) : (
          <div className="flex gap-2">
            <Input
              type="password"
              placeholder="Senha do coordenador"
              value={senha}
              onChange={e => setSenha(e.target.value)}
            />
            <Button onClick={autenticarCoordenador}>Entrar</Button>
          </div>
        )}
      </div>

      {atendentes.map((nome) => (
        <Card key={nome} className="shadow-md">
          <CardContent className="p-4">
            <h2 className="text-xl font-semibold mb-4">{nome}</h2>
            <Tabs defaultValue="semanal">
              <TabsList>
                {categorias.map(cat => (
                  <TabsTrigger key={cat} value={cat}>{cat.toUpperCase()}</TabsTrigger>
                ))}
              </TabsList>

              {categorias.map(cat => (
                <TabsContent key={cat} value={cat} className="mt-4">
                  <div className="space-y-2">
                    <ul className="space-y-1">
                      {(tarefas[nome]?.[cat] || []).map((tarefa, index) => {
                        const atrasada = verificarAtraso(tarefa, cat);
                        return (
                          <li key={index} className="flex items-center gap-2">
                            <Checkbox checked={tarefa.feito} onCheckedChange={() => alternarFeito(nome, cat, index)} />
                            <span className={`${tarefa.feito ? 'line-through text-gray-500' : ''} ${atrasada ? 'text-red-600 font-semibold' : ''}`}>
                              {tarefa.texto} {atrasada && '(Em atraso)'}
                            </span>
                          </li>
                        );
                      })}
                    </ul>
                    {modoCoordenador && (
                      <div className="flex gap-2">
                        <Input
                          placeholder={`Adicionar tarefa ${cat}`}
                          value={novaTarefa}
                          onChange={e => setNovaTarefa(e.target.value)}
                        />
                        <Button onClick={() => adicionarTarefa(nome, cat)}>Adicionar</Button>
                      </div>
                    )}
                  </div>
                </TabsContent>
              ))}
            </Tabs>
          </CardContent>
        </Card>
      ))}
    </div>
  );
};

export default ChecklistAgendamento;
