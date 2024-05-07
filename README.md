- 👋 Hi, I’m @JonaPortillo
- 👀 I’m interested in WEB Programming
- 🌱 I’m currently learning React.js and Node.js
- 👔 I’m working at Circo Studio as Sofware Developer 💻 with Power Platform
- 🎓 I'm studying Software Engineering
- 📫 How to reach me: portillo.jonathan.leonel@gmail.com

<!---
JonaPortillo/JonaPortillo is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
function ayuda(){
	echo "------------ AYUDA - DESCRIPCION DEL SCRIPT -----------"
	echo " "	
	echo "El script ejecuta un sistema de integracion continua. Cada vez que se detecta un cambio en alguno de los archivos del directorio especificado se realiza un backup del directorio y crea un registro en un archivo log"
	echo " "
	echo "------------ AYUDA - FORMATO DEL SRCIPT ------------"
	echo "**Para asegurar el correcto funcionamiento del script, es necesario que antes ejecute los siguientes comandos**"
	echo "I)  sudo apt update"
	echo "II) sudo apt install inotify-tools"
	echo " "
	echo "./ejercicio4.sh [--help | -h] => Muestra ayuda"
	echo "**Parametros de ejecucion**"
	echo "[--directorio | -d] => Directorio que se desea monitorear"
	echo "[--salida | -s] => Directorio donde se guardaran los backups y el archivo de log"
	echo "[--patron | -p] => Patron de caracteres a  buscar en cada modificación/creación"
	echo " "
	echo "**Ejemplo de ejecucion**"
	echo "./ejercicio4.sh -d <path> -s <path> -p 'ejemplo'"
	echo " "
	echo "----------- AYUDA - CIERRE DEFINITIVO DEL SCRIPT ------------"
	echo "1) Para cerrar definitivamente el script, ingrese el comnado ps"
	echo "2) Tome nota de los PID de los procesos con nombre 'ejercicio4.sh' y 'inotifywait'"
	echo "3) Ejecute el comando: kill -10 [PID_de_'ejercicio4.sh']"
	echo "4) Ejecute el comando: kill -10 [PID_de_'inotifywait']"

}

function demonio() {
#Mientras inotifywait este detectando un "cambio" en algun archivo del
#directorio, se ejecuta el siguiente codigo.

	while  archivo=$(inotifywait -e modify,create -r -q --format "%w%f" "$directorio")
	do

		if [[ $archivo != *".goutputstream"* ]]
		then
			coincidePatron=$(grep -c $patron $archivo)
			if [[ "$coincidePatron" != "0" ]]
			then
				cd /tmp 

				fecha=$(date +"%Y%m%d-%H%M%S")
				cp -r $archivo /tmp/"$fecha" 
				
				tar -czf backup.tar.gz "$fecha" 
				
				mv backup.tar.gz "$fecha".tar.gz

				cp "$fecha".tar.gz $salida

				rm -rf "$fecha".tar.gz rm -rf "$fecha"
				cd $salida 
				echo "$fecha: $archivo contiene el patron $patron" >> log.txt
			fi
		fi
	done
}

options=$(getopt -o p:s:d:h --l patron:,salida:,directorio:,help -- "$@" 2> /dev/null) 

if [ "$?" != "0" ] # equivale a: if test "$?" != "0" 
then echo 'Opciones incorrectas' 
	exit 1 
fi

salidaExiste=false

eval set -- "$options" 

while true 
do 
	case "$1" in 
		-s | --salida) 

			salida=$(realpath -m "$2")
			if [[ -d "$salida" ]]; then 
				salidaExiste=true
			fi
			ingresado1=true
			shift 2
			;;
		
	    	-d | --directorio)
			directorio=$(realpath -m "$2")
			shift 2

			ingresado2=true
			;;
		-p | --patron)
			patron="$2"
			shift 2
			
			ingresado3=true
			;;
	    	-h | --help)
			ayuda
			exit 0
			;;
	    	--) # case "--":
			shift
			break
			;;
	    	*) # default: 
			echo "error"
			exit 1
			;;
	esac
done
#------------ EJECUCION EN MODO "DEMONIO" ------------
#Se verifica que se hayan ingresado parametros validos.

if [[ "$ingresado1" == true && "$ingresado2" == true && "$ingresado3" == true ]]; 
then

# Se verifica que el directorio exista y tenga los permisos de lectura.
	if [[ -d "$directorio" && -r "$directorio" ]]; 
	then

		# Se verifica que el directorio a destino exista.
		if [[ "$salidaExiste" == false ]]; then
			mkdir $salida
		fi

		demonio &

	else
		echo "$directorio no es un directorio o no posee los permiso de lectura"
		exit
	fi

else 
	echo "No se ingresaron parametros validos. Pruebe ingresando ./ejercicio4.sh [-h, --help]" 
	exit 
fi
