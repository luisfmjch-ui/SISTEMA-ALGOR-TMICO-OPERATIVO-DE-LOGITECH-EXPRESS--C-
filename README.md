//SISTEMA-ALGOR-TMICO-OPERATIVO-DE-LOGITECH-EXPRESS-c++
// Proyecto Final Algoritmos - LogiTech Express
#include <iostream>
#include <iomanip>
#include <string>
#include <limits>
#include <cstdlib>

using namespace std;

// Constantes del negocio
const double TARIFA_NACIONAL = 50.0;
const double TARIFA_INTERNACIONAL = 250.0;
const double LIMITE_LIBRAS = 10.0;
const double RECARGO_POR_LIBRA = 50.0;
const double INCREMENTO_EXPRESS = 0.30;
const double INCREMENTO_SEGURO = 0.05;
const double LIBRAS_POR_KG = 2.20462;
const double LIMITE_DESCUENTO_MONTO = 1500.0;
const int LIMITE_DESCUENTO_PAQUETES = 10;
const double PORCENTAJE_DESCUENTO = 0.10;

// Prototipos de funciones
void limpiar_buffer();
void finalizar_si_eof();
int leer_opcion(const string& mensaje, int minimo, int maximo);
double leer_decimal(const string& mensaje, bool permitir_cero);
string leer_id();

double calcular_tarifa(int destino, double peso_kg, int servicio, bool seguro,
                       double& base, double& recargo, double& extra_express,
                       double& monto_seguro);
double aplicar_descuento(double total, int cantidad, bool& aplicado);
bool cumple_sla(double horas_estimadas, double horas_reales);

void mostrar_menu();
void registrar_envio(string& id, double& peso_kg, double& costo, bool& es_express);
void modulo_envio_individual();
void modulo_lote();
void modulo_sla();

int main() {
    int opcion = 0;
    while (opcion != 4) {
        cout << fixed << setprecision(2);
        mostrar_menu();
        opcion = leer_opcion("Seleccione una opcion (1-4): ", 1, 4);
        
        switch (opcion) {
            case 1: 
                modulo_envio_individual(); 
                break;
            case 2: 
                modulo_lote(); 
                break;
            case 3: 
                modulo_sla(); 
                break;
            case 4: 
                cout << "\nGracias por usar Logitech Express.\n"; 
                break;
        }
    }
    return 0;
}

// Funciones de validación
void limpiar_buffer() {
    cin.clear();
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
}

void finalizar_si_eof() {
    if (cin.eof()) {
        cout << "\nEntrada finalizada. Cerrando programa.\n";
        exit(0);
    }
}

int leer_opcion(const string& mensaje, int minimo, int maximo) {
    int valor;
    while (true) {
        cout << mensaje;
        if (cin >> valor && valor >= minimo && valor <= maximo) {
            limpiar_buffer();
            return valor;
        }
        finalizar_si_eof();
        limpiar_buffer();
        cout << "Opcion invalida. Ingrese un numero entre " << minimo << " y " << maximo << ".\n";
    }
}

double leer_decimal(const string& mensaje, bool permitir_cero) {
    double valor;
    while (true) {
        cout << mensaje;
        if (cin >> valor && (valor > 0 || (permitir_cero && valor == 0))) {
            limpiar_buffer();
            return valor;
        }
        finalizar_si_eof();
        limpiar_buffer();
        if (permitir_cero)
            cout << "Valor invalido. Debe ser un numero mayor o igual a 0.\n";
        else
            cout << "Valor invalido. Debe ser un numero mayor a 0.\n";
    }
}

string leer_id() {
    string id;
    while (true) {
        cout << "Ingrese ID del paquete: ";
        getline(cin, id);
        finalizar_si_eof();
        if (!id.empty()) return id;
        cout << "El ID no puede estar vacio.\n";
    }
}

// Lógica de cálculo de tarifa
double calcular_tarifa(int destino, double peso_kg, int servicio, bool seguro,
                       double& base, double& recargo, double& extra_express,
                       double& monto_seguro) {
    double peso_lb = peso_kg * LIBRAS_POR_KG;
    
    // Tarifa base (1: Nacional, 2: Internacional)
    if (destino == 1)
        base = TARIFA_NACIONAL;
    else
        base = TARIFA_INTERNACIONAL;
        
    recargo = 0;
    if (peso_lb > LIMITE_LIBRAS) {
        recargo = (peso_lb - LIMITE_LIBRAS) * RECARGO_POR_LIBRA;
    }
    
    double subtotal = base + recargo;
    
    extra_express = 0;
    if (servicio == 2) { // Express
        extra_express = subtotal * INCREMENTO_EXPRESS;
        subtotal += extra_express;
    }
    
    monto_seguro = 0;
    if (seguro) {
        monto_seguro = subtotal * INCREMENTO_SEGURO;
        subtotal += monto_seguro;
    }
    
    return subtotal;
}

double aplicar_descuento(double total, int cantidad, bool& aplicado) {
    if (total > LIMITE_DESCUENTO_MONTO || cantidad > LIMITE_DESCUENTO_PAQUETES) {
        aplicado = true;
        return total * (1.0 - PORCENTAJE_DESCUENTO);
    }
    aplicado = false;
    return total;
}

bool cumple_sla(double horas_estimadas, double horas_reales) {
    return horas_reales <= horas_estimadas;
}

void mostrar_menu() {
    cout << "\n=============================================\n";
    cout << "     LOGITECH EXPRESS - MENU PRINCIPAL     \n";
    cout << "=============================================\n";
    cout << " [1] Registro y Tarificación Individual\n";
    cout << " [2] Procesamiento por Lotes y Estadísticas\n";
    cout << " [3] Diagnóstico y Auditoría de Eficiencia (SLA)\n";
    cout << " [4] Salir\n";
    cout << "---------------------------------------------\n";
}

void modulo_envio_individual() {
    cout << "\n--- MÓDULO 1: REGISTRO Y TARIFICACIÓN ---\n";
    string id = leer_id();
    int destino = leer_opcion("Tipo de destino ([1] Nacional, [2] Internacional): ", 1, 2);
    double peso_kg = leer_decimal("Ingrese el peso en kg (> 0): ", false);
    int servicio = leer_opcion("Tipo de servicio ([1] Estándar, [2] Express): ", 1, 2);
    int req_seguro = leer_opcion("¿Requiere seguro? ([1] Sí, [2] No): ", 1, 2);
    bool seguro = (req_seguro == 1);
    
    double base, recargo, extra_express, monto_seguro;
    double total = calcular_tarifa(destino, peso_kg, servicio, seguro, base, recargo, extra_express, monto_seguro);
    
    cout << "\n---------------- DESGLOSE DE TARIFA ----------------\n";
    cout << " ID Paquete   : " << id << "\n";
    cout << " Tarifa Base  : Q" << base << "\n";
    cout << " Recargo Peso : Q" << recargo << "\n";
    cout << " Extra Express: Q" << extra_express << "\n";
    cout << " Seguro       : Q" << monto_seguro << "\n";
    cout << " TOTAL A PAGAR: Q" << total << "\n";
    cout << "----------------------------------------------------\n";
}

void modulo_lote() {
    cout << "\n--- MÓDULO 2: PROCESAMIENTO POR LOTES ---\n";
    int n = leer_opcion("Ingrese la cantidad de paquetes a procesar en el lote: ", 1, 100);
    
    double total_acumulado = 0;
    double peso_total = 0;
    double mayor_costo = -1;
    string id_mayor = "";
    int express_count = 0;
    
    for (int i = 0; i < n; i++) {
        cout << "\n--- Paquete " << (i + 1) << " de " << n << " ---\n";
        string id = leer_id();
        int destino = leer_opcion("Destino ([1] Nac, [2] Inter): ", 1, 2);
        double peso_kg = leer_decimal("Peso en kg: ", false);
        int servicio = leer_opcion("Servicio ([1] Estándar, [2] Express): ", 1, 2);
        int req_seguro = leer_opcion("Seguro ([1] Sí, [2] No): ", 1, 2);
        bool seguro = (req_seguro == 1);
        
        double base, recargo, extra_express, monto_seguro;
        double costo = calcular_tarifa(destino, peso_kg, servicio, seguro, base, recargo, extra_express, monto_seguro);
        
        total_acumulado += costo;
        peso_total += peso_kg;
        if (costo > mayor_costo) {
            mayor_costo = costo;
            id_mayor = id;
        }
        if (servicio == 2) {
            express_count++;
        }
    }
    
    bool descuento_aplicado = false;
    double gran_total = aplicar_descuento(total_acumulado, n, descuento_aplicado);
    double peso_promedio = peso_total / n;
    
    cout << "\n================ ESTADÍSTICAS DE TURNO ================\n";
    cout << " Total de paquetes procesados : " << n << "\n";
    cout << " Monto acumulado inicial      : Q" << total_acumulado << "\n";
    if (descuento_aplicado) {
        cout << " ¡Descuento corporativo 10% aplicado!\n";
        cout << " Gran Total con Descuento     : Q" << gran_total << "\n";
    } else {
        cout << " Gran Total                   : Q" << gran_total << "\n";
    }
    cout << " Peso promedio procesado      : " << peso_promedio << " kg\n";
    cout << " Paquete con mayor costo      : ID " << id_mayor << " (Q" << mayor_costo << ")\n";
    cout << " Total de envíos Express      : " << express_count << "\n";
    cout << "=======================================================\n";
}

void modulo_sla() {
    cout << "\n--- MÓDULO 3: AUDITORÍA Y SLA ---\n";
    int n = leer_opcion("Ingrese la cantidad de entregas a auditar: ", 1, 100);
    int cumplidas = 0;
    
    for (int i = 0; i < n; i++) {
        cout << "\nEntrega " << (i + 1) << ":\n";
        double est = leer_decimal("  Horas estimadas (SLA): ", false);
        double real = leer_decimal("  Horas reales de tránsito: ", false);
        
        if (cumple_sla(est, real)) {
            cout << "  -> [CUMPLE] con el indicador SLA.\n";
            cumplidas++;
        } else {
            cout << "  -> [NO CUMPLE] con el indicador SLA.\n";
        }
    }
    
    double porcentaje = ((double)cumplidas / n) * 100.0;
    cout << "\n================ RESULTADO DE AUDITORÍA ================\n";
    cout << " Entregas que cumplen SLA : " << cumplidas << " de " << n << "\n";
    cout << " Porcentaje de cumplimiento: " << fixed << setprecision(2) << porcentaje << "%\n";
    cout << "========================================================\n";
}
