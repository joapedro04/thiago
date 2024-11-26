#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <cstdlib>
#include <ctime>
#include <chrono>
#include <thread>

using namespace std;

// Enums para maior organização
enum TipoMagia {
    FOGO,
    GELO,
    RAIO,
    CURA,
    VENENO
};

enum TipoClasse {
    GUERREIRO,
    MAGO,
    ARQUEIRO,
    CAVALEIRO,
    DRUIDA
};

// Estrutura para magias
struct Magia {
    string nome;
    TipoMagia tipo;
    int custo_mana;
    int dano;
    string descricao;
};

// Estrutura para itens melhorada
struct Item {
    string nome;
    int bonus_ataque;
    int bonus_defesa;
    int bonus_vida;
    int bonus_mana;
    string descricao;
    bool equipado;
};

// Estrutura para personagens melhorada
struct Personagem {
    string nome;
    TipoClasse classe;
    int vida;
    int vida_maxima;
    int mana;
    int mana_maxima;
    int ataque;
    int defesa;
    int nivel;
    int experiencia;
    vector<Magia> magias;
    vector<Item> inventario;
    string historia_background;
};

// Estrutura para inimigos
struct Inimigo {
    string nome;
    string descricao;
    int vida;
    int ataque;
    int defesa;
    vector<Magia> magias;
    int exp_dada;
    string dialogo_encontro;
};

// Estrutura para cenários
struct Cenario {
    string nome;
    string descricao;
    vector<Inimigo> inimigos_possiveis;
    vector<Item> itens_possiveis;
    string historia;
};

// Estrutura para salvar o progresso melhorada
struct SaveGame {
    Personagem jogador;
    int cenario_atual;
    int historia_atual;
    vector<Item> inventario;
};

// Função para exibir texto com efeito de digitação
void exibirTextoComEfeito(const string& texto, int velocidade = 30) {
    for (char c : texto) {
        cout << c << flush;
        this_thread::sleep_for(chrono::milliseconds(velocidade));
    }
    cout << endl;
}

// Inicialização das magias
vector<Magia> initMagias(TipoClasse classe) {
    switch(classe) {
        case GUERREIRO:
            return {
                {"Golpe Heroico", FOGO, 15, 25, "Um poderoso golpe carregado de energia"}
            };
        case MAGO:
            return {
                {"Bola de Fogo", FOGO, 20, 30, "Uma explosão de chamas intensas"},
                {"Rajada de Gelo", GELO, 15, 25, "Um jato congelante que pode reduzir a velocidade do inimigo"}
            };
        case ARQUEIRO:
            return {
                {"Flecha Penetrante", RAIO, 10, 20, "Uma flecha carregada de energia elétrica"}
            };
        case CAVALEIRO:
            return {
                {"Investida Sagrada", CURA, 25, -30, "Um ataque que restaura energia do aliado"}
            };
        case DRUIDA:
            return {
                {"Cura da Natureza", CURA, 30, -40, "Restaura uma quantidade significativa de vida"},
                {"Nuvem Venenosa", VENENO, 18, 20, "Causa dano ao longo do tempo"}
            };
        default:
            return {};
    }
}

// Inicialização de personagem melhorada
Personagem inicializarPersonagem(string nome, TipoClasse classe) {
    Personagem p;
    p.nome = nome;
    p.classe = classe;
    p.nivel = 1;
    p.experiencia = 0;
    
    // Atributos base dependendo da classe
    switch(classe) {
        case GUERREIRO:
            p.vida_maxima = 120;
            p.mana_maxima = 50;
            p.ataque = 15;
            p.defesa = 12;
            p.historia_background = "Um valente guerreiro treinado nas artes do combate corpo a corpo.";
            break;
        case MAGO:
            p.vida_maxima = 80;
            p.mana_maxima = 150;
            p.ataque = 10;
            p.defesa = 5;
            p.historia_background = "Um poderoso mago que domina as magias elementais.";
            break;
        case ARQUEIRO:
            p.vida_maxima = 100;
            p.mana_maxima = 70;
            p.ataque = 18;
            p.defesa = 8;
            p.historia_background = "Um arqueiro preciso e ágil, mestre do arco e flecha.";
            break;
        case CAVALEIRO:
            p.vida_maxima = 150;
            p.mana_maxima = 60;
            p.ataque = 12;
            p.defesa = 15;
            p.historia_background = "Um nobre cavaleiro protegido por uma armadura sagrada.";
            break;
        case DRUIDA:
            p.vida_maxima = 90;
            p.mana_maxima = 120;
            p.ataque = 11;
            p.defesa = 9;
            p.historia_background = "Um guardião da natureza com poderes místicos de cura e destruição.";
            break;
    }
    
    // Inicializa vida e mana completas
    p.vida = p.vida_maxima;
    p.mana = p.mana_maxima;
    
    // Inicializa magias específicas da classe
    p.magias = initMagias(classe);
    
    return p;
}
// Adicione estas funções após o código anterior

// Função para gerenciar combate
bool combate(Personagem& jogador, Inimigo& inimigo) {
    exibirTextoComEfeito("Um " + inimigo.nome + " apareceu! " + inimigo.dialogo_encontro);
    
    while (jogador.vida > 0 && inimigo.vida > 0) {
        // Turno do jogador
        cout << "\nSua vida: " << jogador.vida << "/" << jogador.vida_maxima 
             << " | Mana: " << jogador.mana << "/" << jogador.mana_maxima << endl;
        cout << "Inimigo " << inimigo.nome << " vida: " << inimigo.vida << endl;
        
        cout << "\nEscolha sua ação:" << endl;
        cout << "1. Ataque básico" << endl;
        cout << "2. Usar magia" << endl;
        
        int escolha;
        cin >> escolha;
        
        if (escolha == 1) {
            // Ataque básico
            int dano = max(0, jogador.ataque - inimigo.defesa);
            inimigo.vida -= dano;
            exibirTextoComEfeito("Você causa " + to_string(dano) + " de dano!");
        } else if (escolha == 2) {
            // Escolher magia
            cout << "Magias disponíveis:" << endl;
            for (size_t i = 0; i < jogador.magias.size(); ++i) {
                cout << i+1 << ". " << jogador.magias[i].nome 
                     << " (Custo: " << jogador.magias[i].custo_mana 
                     << " | Dano: " << jogador.magias[i].dano << ")" << endl;
            }
            
            int escolha_magia;
            cin >> escolha_magia;
            
            if (escolha_magia > 0 && escolha_magia <= jogador.magias.size()) {
                Magia& magia_escolhida = jogador.magias[escolha_magia-1];
                
                if (jogador.mana >= magia_escolhida.custo_mana) {
                    jogador.mana -= magia_escolhida.custo_mana;
                    
                    if (magia_escolhida.dano > 0) {
                        int dano_magia = max(0, magia_escolhida.dano - inimigo.defesa);
                        inimigo.vida -= dano_magia;
                        exibirTextoComEfeito(magia_escolhida.nome + " causa " + to_string(dano_magia) + " de dano!");
                    } else {
                        // Efeito de cura
                        jogador.vida = min(jogador.vida_maxima, jogador.vida - magia_escolhida.dano);
                        exibirTextoComEfeito(magia_escolhida.nome + " restaura " + to_string(-magia_escolhida.dano) + " de vida!");
                    }
                } else {
                    exibirTextoComEfeito("Mana insuficiente para usar esta magia!");
                }
            }
        }
        
        // Turno do inimigo
        if (inimigo.vida > 0) {
            int dano_inimigo = max(0, inimigo.ataque - jogador.defesa);
            jogador.vida -= dano_inimigo;
            exibirTextoComEfeito(inimigo.nome + " causa " + to_string(dano_inimigo) + " de dano!");
        }
    }
    
    if (jogador.vida <= 0) {
        exibirTextoComEfeito("Você foi derrotado!");
        return false;
    } else {
        exibirTextoComEfeito("Você derrotou " + inimigo.nome + "!");
        // Ganhar experiência
        jogador.experiencia += inimigo.exp_dada;
        return true;
    }
}

// Função para gerenciar inventário
void menuInventario(Personagem& jogador) {
    while (true) {
        cout << "\n--- Inventário ---" << endl;
        cout << "Itens:" << endl;
        
        if (jogador.inventario.empty()) {
            cout << "Inventário vazio" << endl;
        } else {
            for (size_t i = 0; i < jogador.inventario.size(); ++i) {
                Item& item = jogador.inventario[i];
                cout << i+1 << ". " << item.nome 
                     << (item.equipado ? " (Equipado)" : "") << endl;
                cout << "   Bônus - Ataque: " << item.bonus_ataque 
                     << " | Defesa: " << item.bonus_defesa 
                     << " | Vida: " << item.bonus_vida 
                     << " | Mana: " << item.bonus_mana << endl;
            }
        }
        
        cout << "\nEscolha uma ação:" << endl;
        cout << "1. Equipar/Desequipar item" << endl;
        cout << "2. Ver detalhes do item" << endl;
        cout << "3. Voltar" << endl;
        
        int escolha;
        cin >> escolha;
        
        if (escolha == 1) {
            if (!jogador.inventario.empty()) {
                cout << "Escolha o item para equipar/desequipar: ";
                int item_escolha;
                cin >> item_escolha;
                
                if (item_escolha > 0 && item_escolha <= jogador.inventario.size()) {
                    Item& item = jogador.inventario[item_escolha-1];
                    item.equipado = !item.equipado;
                    
                    // Ajustar atributos do personagem
                    jogador.ataque += item.equipado ? item.bonus_ataque : -item.bonus_ataque;
                    jogador.defesa += item.equipado ? item.bonus_defesa : -item.bonus_defesa;
                    jogador.vida_maxima += item.equipado ? item.bonus_vida : -item.bonus_vida;
                    jogador.mana_maxima += item.equipado ? item.bonus_mana : -item.bonus_mana;
                }
            }
        } else if (escolha == 2) {
            if (!jogador.inventario.empty()) {
                cout << "Escolha o item para ver detalhes: ";
                int item_detalhes;
                cin >> item_detalhes;
                
                if (item_detalhes > 0 && item_detalhes <= jogador.inventario.size()) {
                    Item& item = jogador.inventario[item_detalhes-1];
                    cout << "Nome: " << item.nome << endl;
                    cout << "Descrição: " << item.descricao << endl;
                    cout << "Bônus de Ataque: " << item.bonus_ataque << endl;
                    cout << "Bônus de Defesa: " << item.bonus_defesa << endl;
                    cout << "Bônus de Vida: " << item.bonus_vida << endl;
                    cout << "Bônus de Mana: " << item.bonus_mana << endl;
                }
            }
        } else if (escolha == 3) {
            break;
        }
    }
}

// Função de progressão de personagem
void verificarProgressao(Personagem& jogador) {
    int exp_necessaria = jogador.nivel * 100;
    
    if (jogador.experiencia >= exp_necessaria) {
        jogador.nivel++;
        jogador.experiencia -= exp_necessaria;
        
        // Aumentar atributos
        jogador.vida_maxima += 20;
        jogador.mana_maxima += 15;
        jogador.ataque += 3;
        jogador.defesa += 2;
        
        jogador.vida = jogador.vida_maxima;
        jogador.mana = jogador.mana_maxima;
        
        exibirTextoComEfeito("PARABÉNS! Você subiu para o nível " + to_string(jogador.nivel) + "!");
        exibirTextoComEfeito("Seus atributos foram aumentados!");
    }
}

// Função para salvar o jogo
void salvarJogo(const Personagem& jogador, int cenario_atual) {
    ofstream arquivo("save.dat", ios::binary);
    
    if (arquivo.is_open()) {
        // Salvar informações do jogador
        arquivo.write(reinterpret_cast<const char*>(&jogador), sizeof(Personagem));
        arquivo.write(reinterpret_cast<const char*>(&cenario_atual), sizeof(int));
        
        arquivo.close();
        exibirTextoComEfeito("Jogo salvo com sucesso!");
    } else {
        exibirTextoComEfeito("Erro ao salvar o jogo.");
    }
}

// Função para carregar o jogo
bool carregarJogo(Personagem& jogador, int& cenario_atual) {
    ifstream arquivo("save.dat", ios::binary);
    
    if (arquivo.is_open()) {
        arquivo.read(reinterpret_cast<char*>(&jogador), sizeof(Personagem));
        arquivo.read(reinterpret_cast<char*>(&cenario_atual), sizeof(int));
        
        arquivo.close();
        exibirTextoComEfeito("Jogo carregado com sucesso!");
        return true;
    } else {
        exibirTextoComEfeito("Nenhum jogo salvo encontrado.");
        return false;
    }
}
vector<Cenario> initCenarios() {
    vector<Cenario> cenarios = {
        // Cenário 1: Floresta Antiga
        {
            "Floresta Antiga",
            "Uma floresta mística coberta por uma névoa densa. Árvores seculares se erguem como gigantes silenciosos.",
            // Inimigos possíveis
            {
                {"Goblin Guardião", "Um pequeno e astuto guardião da floresta", 50, 10, 5, 
                    {{"Golpe de Clava", FOGO, 10, 15, "Um ataque brutal com uma clava ancestral"}}, 
                    30, "Pare agora mesmo, intruso!"},
                {"Lobo das Sombras", "Um lobo negro com olhos brilhantes", 40, 12, 3, 
                    {{"Mordida Feroz", VENENO, 8, 18, "Uma mordida venenosa"}}, 
                    25, "Uivo ameaçador"}
            },
            // Itens possíveis
            {
                {"Amuleto da Floresta", 0, 5, 20, 10, "Um amuleto que aumenta a conexão com a natureza", false},
                {"Espada do Guardião", 15, 0, 0, 0, "Uma espada antiga com inscrições élficas", false}
            },
            "Rumores dizem que uma antiga tribo élfica guardava segredos profundos nesta floresta."
        },
        
        // Cenário 2: Ruínas do Deserto
        {
            "Ruínas do Deserto",
            "Ruínas de uma antiga civilização soterradas pela areia. Pirâmides parcialmente enterradas se destacam no horizonte.",
            // Inimigos possíveis
            {
                {"Múmia Guerreira", "Uma múmia antiga protegendo os tesouros de seu reino", 70, 15, 8, 
                    {{"Maldição Ancestral", RAIO, 20, 25, "Uma maldição que drena a energia vital"}}, 
                    50, "Ouse entrar em meu domínio e seja amaldiçoado!"},
                {"Escorpião Gigante", "Um escorpião monstruoso adaptado ao deserto", 60, 12, 6, 
                    {{"Picada Venenosa", VENENO, 15, 20, "Um veneno mortal que corrói a carne"}}, 
                    40, "Sons de pinças se abrindo"}
            },
            // Itens possíveis
            {
                {"Bússola do Explorador", 5, 3, 0, 15, "Uma bússola mágica que nunca falha", false},
                {"Amuleto de Proteção Solar", 0, 10, 30, 0, "Protege contra o calor extremo do deserto", false}
            },
            "Lendas contam de um tesouro escondido nas profundezas destas ruínas, guardado por criaturas antigas."
        },
        
        // Cenário 3: Cavernas de Gelo
        {
            "Cavernas de Gelo",
            "Um labirinto de cavernas cristalinas, onde o gelo forma padrões hipnóticos e o frio é cortante como uma lâmina.",
            // Inimigos possíveis
            {
                {"Golem de Gelo", "Uma criatura colossal feita de gelo e rocha", 90, 18, 12, 
                    {{"Rajada Congelante", GELO, 25, 30, "Um sopro que congela instantaneamente"}}, 
                    60, "Intruso... será transformado em gelo!"},
                {"Yeti das Neves", "Uma criatura mitológica das montanhas geladas", 80, 16, 7, 
                    {{"Rugido Congelante", GELO, 20, 25, "Um rugido que paralisa pelo medo"}}, 
                    45, "Rugido ensurdecedor"}
            },
            // Itens possíveis
            {
                {"Casaco de Pele de Yeti", 0, 15, 50, 0, "Uma proteção perfeita contra o frio extremo", false},
                {"Machado de Gelo", 20, 5, 0, 10, "Uma arma antiga forjada nas cavernas", false}
            },
            "Dizem que nestas cavernas habita um antigo guardião, protetor de segredos gelados e tesouros escondidos."
        }
    };
    
    return cenarios;
}

// Função para explorar cenário
bool explorarCenario(Personagem& jogador, Cenario& cenario) {
    exibirTextoComEfeito("Você entra em: " + cenario.nome);
    exibirTextoComEfeito(cenario.descricao);
    exibirTextoComEfeito(cenario.historia);
    
    // Chance de encontrar inimigo ou item
    int encontro = rand() % 3;
    
    if (encontro == 0 && !cenario.inimigos_possiveis.empty()) {
        // Encontro com inimigo
        int inimigo_index = rand() % cenario.inimigos_possiveis.size();
        Inimigo& inimigo_encontrado = cenario.inimigos_possiveis[inimigo_index];
        
        bool resultado_combate = combate(jogador, inimigo_encontrado);
        
        if (resultado_combate) {
            // Chance de ganhar item após derrotar inimigo
            if (!cenario.itens_possiveis.empty() && rand() % 2 == 0) {
                int item_index = rand() % cenario.itens_possiveis.size();
                Item item_ganho = cenario.itens_possiveis[item_index];
                
                exibirTextoComEfeito("Você encontrou: " + item_ganho.nome);
                jogador.inventario.push_back(item_ganho);
            }
            
            verificarProgressao(jogador);
        }
        
        return resultado_combate;
    } else if (encontro == 1 && !cenario.itens_possiveis.empty()) {
        // Encontro com item
        int item_index = rand() % cenario.itens_possiveis.size();
        Item item_encontrado = cenario.itens_possiveis[item_index];
        
        exibirTextoComEfeito("Você encontrou um item: " + item_encontrado.nome);
        exibirTextoComEfeito(item_encontrado.descricao);
        
        char escolha;
        cout << "Deseja pegar o item? (s/n): ";
        cin >> escolha;
        
        if (escolha == 's' || escolha == 'S') {
            jogador.inventario.push_back(item_encontrado);
            exibirTextoComEfeito("Item adicionado ao inventário!");
        }
        
        return true;
    } else {
        // Exploração sem encontros
        exibirTextoComEfeito("Você explora o cenário, mas não encontra nada de especial esta vez.");
        return true;
    }
}

// Modificar a função menuPrincipal para incluir os cenários
void menuPrincipal() {
    Personagem jogador;
    int cenario_atual = 0;
    vector<Cenario> cenarios = initCenarios();
    
    while (true) {
        cout << "\n--- RPG Terminal ---" << endl;
        cout << "1. Novo Jogo" << endl;
        cout << "2. Carregar Jogo" << endl;
        cout << "3. Sair" << endl;
        
        int escolha;
        cin >> escolha;
        
        if (escolha == 1) {
            string nome;
            int classe;
            
            cout << "Digite o nome do seu personagem: ";
            cin >> nome;
            
            cout << "Escolha sua classe:" << endl;
            cout << "1. Guerreiro" << endl;
            cout << "2. Mago" << endl;
            cout << "3. Arqueiro" << endl;
            cout << "4. Cavaleiro" << endl;
            cout << "5. Druida" << endl;
            
            cin >> classe;
            
            jogador = inicializarPersonagem(nome, static_cast<TipoClasse>(classe-1));
        } else if (escolha == 2) {
            if (!carregarJogo(jogador, cenario_atual)) {
                continue;
            }
        } else if (escolha == 3) {
            break;
        }
        
        // Loop principal do jogo
        while (true) {
            cout << "\n--- Opções ---" << endl;
            cout << "1. Explorar" << endl;
            cout << "2. Inventário" << endl;
            cout << "3. Status" << endl;
            cout << "4. Salvar Jogo" << endl;
            cout << "5. Sair do Jogo" << endl;
            
            cin >> escolha;
            
            if (escolha == 1) {
                // Exploração de cenários
                if (cenario_atual >= cenarios.size()) {
                    cenario_atual = 0;
                }
                
                bool resultado_exploracao = explorarCenario(jogador, cenarios[cenario_atual]);
                
                if (resultado_exploracao) {
                    cenario_atual++;
                }
            } else if (escolha == 2) {
                menuInventario(jogador);
            } else if (escolha == 3) {
                cout << "Nome: " << jogador.nome << endl;
                cout << "Classe: " << jogador.classe << endl;
                cout << "Nível: " << jogador.nivel << endl;
                cout << "Experiência: " << jogador.experiencia << endl;
                cout << "Vida: " << jogador.vida << "/" << jogador.vida_maxima << endl;
                cout << "Mana: " << jogador.mana << "/" << jogador.mana_maxima << endl;
                cout << "Ataque: " << jogador.ataque << endl;
                cout << "Defesa: " << jogador.defesa << endl;
            } else if (escolha == 4) {
                salvarJogo(jogador, cenario_atual);
            } else if (escolha == 5) {
                break;
            }
        }
    }
}
int main() {
    srand(time(nullptr));
    menuPrincipal();
    return 0;
}
