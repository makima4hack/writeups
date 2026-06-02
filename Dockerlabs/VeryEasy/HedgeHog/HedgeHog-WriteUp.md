# WriteUp - HedgeHog

**Objetivos**

* Obtener acceso al sistema mediante SSH.
* Realizar una escalada de privilegios hasta `root`.

---

# 🔎 Reconocimiento

Se realiza un escaneo inicial con:

```bash
nmap -sS -Pn -n -sC -sV --top-ports 40 --open 172.17.0.2
```

> [!TIP]
> Se emplean parámetros orientados al sigilo, detección de versiones y enumeración de servicios, evitando además el ping y la resolución DNS. También se limitan las pruebas a los 40 puertos más comunes y únicamente se muestran aquellos que se encuentran abiertos.

![step\_01](Screenshots/step_01.png)

El escaneo revela la presencia de los servicios `SSH` y `HTTP` en los puertos `22` y `80` respectivamente.

## Acceso al servicio web

Con la información obtenida mediante nmap, se accede al servicio web, donde aparece la palabra `tails`.

![step\_02](Screenshots/step_02.png)

> [!NOTE]
> Se deduce que `tails` podría corresponder a un usuario válido del servicio SSH.

---

# 💣 Explotación

Con la información recopilada, se realiza un ataque de fuerza bruta con Hydra contra el usuario `tails`.

```bash
hydra -l tails -P /usr/share/dict/rockyou.txt ssh://172.17.0.2
```

> [!NOTE]
> El ataque tarda demasiado tiempo en producir resultados. Debido al nombre del usuario (`tails`), se plantea la posibilidad de que la contraseña se encuentre al final del diccionario, por lo que se decide invertir el contenido de `rockyou.txt`.

> [!IMPORTANT]
> Inicialmente se prueba el siguiente comando:
>
> ```bash
> tac /usr/share/dict/rockyou.txt > reverse_rockyou.txt
> ```
>
> Sin embargo, se detectan espacios en blanco en algunas entradas del diccionario, por lo que finalmente se emplea:
>
> ```bash
> tac /usr/share/dict/rockyou.txt | tr -d ' ' > reverse_rockyou.txt
> ```

![step\_03](Screenshots/step_03.png)

A continuación, se ejecuta nuevamente Hydra utilizando el diccionario invertido.

![step\_05](Screenshots/step_05.png)

El ataque permite obtener las credenciales del usuario `tails`, cuya contraseña es:

```text
3117548331
```

---

# 🔑 Acceso y Escalada de privilegios

Con las credenciales obtenidas, se intenta acceder por SSH al sistema.

![step\_06](Screenshots/step_06.png)

El acceso se realiza correctamente como el usuario `tails`.

A continuación, se ejecuta `sudo -l` para comprobar los privilegios sudo disponibles.

![step\_07](Screenshots/step_07.png)

Se comprueba que el usuario `tails` no dispone de privilegios sudo directos, pero sí puede ejecutar comandos como el usuario `sonic`.

Tras identificar este comportamiento, se prueba la ejecución de comandos como `sonic` mediante `sudo -u`.

![step\_08](Screenshots/step_08.png)

```bash
sudo -u sonic sudo su
```

La ejecución permite obtener una shell con privilegios de `root`.

> [!TIP]
> Durante la enumeración también se identifica un archivo perteneciente al usuario `sonic` que contiene sus credenciales.

---

# 🗒️ Lecciones aprendidas

* Uso de Hydra para ataques de fuerza bruta sobre servicios SSH.
* Importancia del razonamiento contextual durante la explotación.
* Manipulación de diccionarios utilizando herramientas como `tac` y `tr`.
* Escalada de privilegios mediante `sudo -u`.
