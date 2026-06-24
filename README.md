# TP-Organizaci-n-Empresarial-integrador
import pandas as pd

ESTADO_INICIO = "INICIO"
ESTADO_ESPERANDO_LEGAJO = "ESPERANDO_LEGAJO"
ESTADO_VALIDANDO_LEGAJO = "VALIDANDO_LEGAJO"
ESTADO_CONSULTANDO_SALDO = "CONSULTANDO_SALDO"
ESTADO_ESPERANDO_DIAS = "ESPERANDO_DIAS"
ESTADO_VALIDANDO_DIAS = "VALIDANDO_DIAS"
ESTADO_FINALIZADO = "FINALIZADO"

try:
    empleados = pd.read_excel("empleados.xlsx")
except FileNotFoundError:
    print("Error: No se encontró el archivo empleados.xlsx")
    exit()

estado = ESTADO_INICIO

print("=" * 50)
print("CHATBOT DE GESTIÓN DE VACACIONES")
print("=" * 50)

estado = ESTADO_ESPERANDO_LEGAJO

while estado != ESTADO_FINALIZADO:

    if estado == ESTADO_ESPERANDO_LEGAJO:
        legajo = input("\nIngrese su legajo: ")
        estado = ESTADO_VALIDANDO_LEGAJO

    elif estado == ESTADO_VALIDANDO_LEGAJO:

        if not legajo.isdigit():
            print("Error: El legajo debe ser numérico.")
            estado = ESTADO_ESPERANDO_LEGAJO
            continue

        legajo = int(legajo)

        empleado = empleados[empleados["Legajo"] == legajo]

        if empleado.empty:
            print("Error: Legajo inexistente.")
            estado = ESTADO_FINALIZADO
        else:
            estado = ESTADO_CONSULTANDO_SALDO

    elif estado == ESTADO_CONSULTANDO_SALDO:

        nombre = empleado.iloc[0]["Nombre"]
        saldo = int(empleado.iloc[0]["Dias_Disponibles"])

        print(f"\nBienvenido/a {nombre}")
        print(f"Días disponibles: {saldo}")

        if saldo <= 0:
            print("No posee días disponibles.")
            estado = ESTADO_FINALIZADO
        else:
            estado = ESTADO_ESPERANDO_DIAS

    elif estado == ESTADO_ESPERANDO_DIAS:

        dias = input("Ingrese la cantidad de días que desea solicitar: ")
        estado = ESTADO_VALIDANDO_DIAS

    elif estado == ESTADO_VALIDANDO_DIAS:

        if not dias.isdigit():
            print("Error: Debe ingresar un número.")
            estado = ESTADO_ESPERANDO_DIAS
            continue

        dias = int(dias)

        if dias <= 0:
            print("Error: Debe solicitar al menos un día.")
            estado = ESTADO_ESPERANDO_DIAS

        elif dias > saldo:
            print(
                f"Solicitud rechazada. "
                f"Solo dispone de {saldo} días."
            )
            estado = ESTADO_FINALIZADO

        else:
            nuevo_saldo = saldo - dias

            empleados.loc[
                empleados["Legajo"] == legajo,
                "Dias_Disponibles"
            ] = nuevo_saldo

            empleados.to_excel(
                "empleados.xlsx",
                index=False
            )

            print("\nSolicitud aprobada.")
            print(f"Días solicitados: {dias}")
            print(f"Nuevo saldo: {nuevo_saldo}")

            estado = ESTADO_FINALIZADO

print("\nProceso finalizado.")
