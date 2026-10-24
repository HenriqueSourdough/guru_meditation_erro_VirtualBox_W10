📚 Relatório Técnico: Resolução do Erro Guru Meditation no VirtualBox (Windows 10 Guest em Host AMD Ryzen)
📌 1. Visão Geral do Problema
Sintoma: Ocorria uma falha crítica imediata de sistema (Guru Meditation) no momento do boot ao tentar iniciar uma Máquina Virtual (VM) do Windows 10 no VirtualBox. Em contrapartida, outras VMs (como opNsense e Ubuntu Server) inicializavam normalmente no mesmo ambiente hospedeiro.

Causa-Raiz: Conflito de tradução de instruções de CPU de baixo nível entre o motor de virtualização do VirtualBox, a arquitetura do processador hospedeiro (AMD Ryzen) e o kernel de inicialização do instalador do Windows 10. O instalador do Windows exige um conjunto específico de instruções de hardware no arranque; ao tentar intermediar essa comunicação com a arquitetura Ryzen sem uma máscara adequada, a camada de emulação do VirtualBox falhava ao traduzir os comandos, resultando no encerramento forçado da VM por segurança.

🔍 2. Diagnóstico e Isolamento da Falha
Análise de Logs (VBox.log): Os registros de execução apontavam assimetria no uso de núcleos de processamento e interrupções de travamento em chamadas de instrução no momento do salto do kernel.

Isolamento por Comparação (O Teste Linux): A criação e execução bem-sucedida de uma VM com Ubuntu Server serviu como ponto de virada no diagnóstico. Como o kernel Linux lida de forma mais flexível com variações de arquitetura de CPU, ele iniciou perfeitamente. Isso provou que:

A virtualização de hardware na BIOS/UEFI (AMD-V) estava ativa e funcional.

A instalação do VirtualBox e os drivers do sistema operacional hospedeiro estavam íntegros.

O problema era estritamente a incompatibilidade entre a forma como o VirtualBox repassava as instruções nativas do AMD Ryzen para o instalador do Windows 10.

Descarte de Hipóteses: Foram descartadas falhas de configuração em EFI/BIOS, modos de paravirtualização (Hyper-V/KVM), tipos de placa de rede e resíduos de instalação desatendida (Unattended).

🛠️ 3. A Solução Definitiva (Mascaramento de CPU)
Para contornar a limitação de tradução de instruções da arquitetura Ryzen no instalador do Windows 10, aplicou-se o mascaramento do perfil do processador via interface de linha de comando do VirtualBox (VBoxManage).

Ao definir um perfil genérico e padronizado de CPU, o VirtualBox passou a apresentar ao Windows 10 um conjunto de instruções previamente mapeado que o motor de virtualização consegue traduzir sem falhas para o processador físico.

Comando Executado (Prompt de Comando como Administrador):

DOS
cd /d "C:\Program Files\Oracle\VirtualBox"
VBoxManage modifyvm "Win10" --cpu-profile "Intel Core i7-6700K"
(Nota: Substitua "Win10" pelo nome exato atribuído à sua VM).

✅ 4. Resultado e Conclusão
Após a execução do comando, a tradução de instruções de CPU passou a ocorrer de forma estável. O instalador do Windows 10 superou a tela de boot sem disparar o Guru Meditation, permitindo a conclusão normal do processo de instalação do sistema operacional.

💡 5. Lições Aprendidas
Processadores AMD Ryzen possuem uma implementação de instruções de virtualização que, em versões específicas do VirtualBox, podem exigir ajuste na assinatura ou perfil de CPU para sistemas convidados da Microsoft.

Testar distribuições Linux é uma excelente prática de diagnóstico para isolar se uma falha no VirtualBox é global (sistema/hardware) ou específica do sistema operacional convidado (Guest OS).
