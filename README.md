#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>


#ifdef _WIN32
#include <windows.h>
#define PAUSA(segundos) Sleep((segundos) * 1000)
#else
#include <unistd.h>
#define PAUSA(segundos) sleep(segundos)
#endif


typedef struct {
    char nome[30];
    float taxa_corretagem;
    int suporte_api; // 1 = Sim, 0 = Nao (Essencial para automacao)
} Corretora;


typedef struct {
    float saldo_reais;
    int quantidade_acoes;
    float preco_medio_compra;
} Carteira;


    
    for (int i = 0; i < total; i++) {
        if (lista[i].suporte_api == 1 && lista[i].taxa_corretagem < melhor.taxa_corretagem) {
            melhor = lista[i];
        }
    }
    return melhor;
}


float obter_preco_api(float preco_anterior) {
    // Gera uma variacao aleatoria entre -1.50 e +1.50 para simular o mercado
    float variacao = ((rand() % 301) - 150) / 100.0;
    float novo_preco = preco_anterior + variacao;
    return (novo_preco < 10.0) ? 10.0 : novo_preco;
}


void enviar_ordem_api(const char* corretora, const char* tipo, const char* ativo, int qtd, float preco) {
    printf("   >>> [API %s] ENVIANDO ORDEM: %s %d x %s a R$ %.2f <<<\n", 
           corretora, tipo, qtd, ativo, preco);
}

int main() {
    srand(time(NULL)); // Inicializa gerador aleatorio para simular mercado

    
    Corretora corretoras[] = {
        {"XP Investimentos", 4.90, 1},
        {"Rico",             0.00, 0}, // Sem API aberta para robo proprio
        {"BTG Pactual",      2.50, 1}, // Melhor opcao com API
        {"Clear Corretora",  0.00, 0}
    };

    
    
    if (escolhida.suporte_api == 0) {
        printf("Erro: Nenhuma corretora com suporte a API disponivel.\n");
        return 1;
    }

    printf("====================================================\n");
    printf("BOT INICIADO | Corretora Conectada: %s\n", escolhida.nome);
    printf("Taxa por operacao: R$ %.2f\n", escolhida.taxa_corretagem);
    printf("====================================================\n\n");

    
    Carteira minha_carteira = {1000.00, 0, 0.0}; // Comeca com R$ 1.000,00
    const char* ativo = "PETR4";
    float preco_atual = 35.00;
    float alvo_compra = 34.00; // Compra se cair abaixo de R$ 34,00
    float alvo_lucro = 1.05;   // Vende se subir 5% acima do preco pago

    
    for (int ciclo = 1; ciclo <= 10; ciclo++) {
        preco_atual = obter_preco_api(preco_atual);
        printf("[Ciclo %02d] %s: R$ %.2f | Saldo: R$ %.2f | Acoes: %d\n", 
               ciclo, ativo, preco_atual, minha_carteira.saldo_reais, minha_carteira.quantidade_acoes);

        
        if (minha_carteira.quantidade_acoes == 0 && preco_atual <= alvo_compra) {
            int qtd_comprar = (int)((minha_carteira.saldo_reais - escolhida.taxa_corretagem) / preco_atual);
            
            if (qtd_comprar > 0) {
                float custo_total = (qtd_comprar * preco_atual) + escolhida.taxa_corretagem;
                minha_carteira.saldo_reais -= custo_total;
                minha_carteira.quantidade_acoes = qtd_comprar;
                minha_carteira.preco_medio_compra = preco_atual;

                enviar_ordem_api(escolhida.nome, "COMPRA", ativo, qtd_comprar, preco_atual);
            }
        }

        else if (minha_carteira.quantidade_acoes > 0 && preco_atual >= (minha_carteira.preco_medio_compra * alvo_lucro)) {
            int qtd_vender = minha_carteira.quantidade_acoes;
            float valor_venda = (qtd_vender * preco_atual) - escolhida.taxa_corretagem;
            
            minha_carteira.saldo_reais += valor_venda;
            minha_carteira.quantidade_acoes = 0;

            enviar_ordem_api(escolhida.nome, "VENDA", ativo, qtd_vender, preco_atual);
            printf("   *** LUCRO REALIZADO! Novo Saldo: R$ %.2f ***\n", minha_carteira.saldo_reais);
        }

        
    }

    printf("\n====================================================\n");
    printf("FIM DOS CICLOS | Saldo Final: R$ %.2f | Acoes em carteira: %d\n", 
           minha_carteira.saldo_reais, minha_carteira.quantidade_acoes);
    printf("====================================================\n");

    return 0;
}
