import requests
import urllib.parse

route_url = "https://graphhopper.com/api/1/route?"
key = "31b32dae-9661-460e-bf62-24b956427e24"  #Aquí debes reemplazar con la clave API entregada por graphhopper.com


def geocoding(location, key):
    geocode_url = "https://graphhopper.com/api/1/geocode?"
    
    while location == "":
        location = input("Ingrese ubicación otra vez: ")
    
    url = geocode_url + urllib.parse.urlencode({"q":location, "limit": "1", "key":key})
    
    replydata = requests.get(url)
    json_data = replydata.json()
    json_status = replydata.status_code
    
    if json_status == 200 and len(json_data["hits"]) != 0:
        lat = json_data["hits"][0]["point"]["lat"]
        lng = json_data["hits"][0]["point"]["lng"]
        name = json_data["hits"][0]["name"]
        value = json_data["hits"][0]["osm_value"]
        
        if "country" in json_data["hits"][0]:
            country = json_data["hits"][0]["country"]
        else:
            country=""
            
        if "state" in json_data["hits"][0]:
            state = json_data["hits"][0]["state"]
        else:
            state=""
            
        if len(state) !=0 and len(country) !=0:
            new_loc = name + ", " + state + ", " + country
        elif len(state) !=0:
            new_loc = name + ", " + country
        else:
            new_loc = name
            
        print("URL geocode para " + new_loc + " (Tipo: " + value + "):")
        print(url)
        return json_status, lat, lng, new_loc
    else:
        lat=lng="null"
        new_loc=location
        if json_status != 200:
            print("Estado API: " + str(json_status))
            print("Error: " + json_data["message"])
        return json_status, lat, lng, new_loc

print("\nPerfiles de vehículo en Graphhopper:")
print("auto, bicicleta, a_pie")  

while True:
    vehicle = input("\nIngrese La opcion para viajar. ingrese (s) o (salir) para finalizar: ")
    
    if vehicle.lower() in ["s", "salir"]:
        print("Programa terminado.")
        break

    profiles_esp = ["auto", "bicicleta", "a_pie"]
    profiles_api = ["car", "bike", "foot"]
    
    if vehicle in profiles_esp:
        if vehicle == "auto":
            vehicle_api = "car"
        elif vehicle == "bicicleta":
            vehicle_api = "bike"
        else:  
            vehicle_api = "foot"
    else:
        vehicle_api = "car"
        print("No valido.")
    
    loc1 = input("Ubicación de inicio: ")
    if loc1.lower() in ["s", "salir"]:
        break
        
    orig = geocoding(loc1, key)
    
    loc2 = input("Destino: ")
    if loc2.lower() in ["s", "salir"]:
        break
        
    dest = geocoding(loc2, key)
    
    print("="*50)
    
    if orig[0] == 200 and dest[0] == 200:
        op="&point="+str(orig[1])+"%2C"+str(orig[2])
        dp="&point="+str(dest[1])+"%2C"+str(dest[2])
        
        paths_url = route_url + urllib.parse.urlencode({"key":key, "vehicle":vehicle_api}) + op + dp
        paths_status = requests.get(paths_url).status_code
        paths_data = requests.get(paths_url).json()
        
        print("Estado API Routing: " + str(paths_status))
        print("URL Routing: " + paths_url)
        print("="*50)
        
        if vehicle_api == "car":
            veh_mostrar = "auto"
        elif vehicle_api == "bike":
            veh_mostrar = "bicicleta"
        else:
            veh_mostrar = "a pie"
            
        print("Direcciones de " + orig[3] + " a " + dest[3] + " en " + veh_mostrar)
        print("="*50)
        
        if paths_status == 200:
            miles = (paths_data["paths"][0]["distance"])/1000/1.61
            km = (paths_data["paths"][0]["distance"])/1000
            sec = int(paths_data["paths"][0]["time"]/1000%60)
            min = int(paths_data["paths"][0]["time"]/1000/60%60)
            hr = int(paths_data["paths"][0]["time"]/1000/60/60)
            
            print(f"Distancia Recorrida: {miles:.2f} millas / {km:.2f} km")
            print(f"Duración Viaje: {hr:02d}:{min:02d}:{sec:02d}")
            print("="*50)
            
            for i in range(len(paths_data["paths"][0]["instructions"])):
                path = paths_data["paths"][0]["instructions"][i]["text"]
                distance = paths_data["paths"][0]["instructions"][i]["distance"]
                
                if "Continue onto" in path:
                    path = path.replace("Continue onto", "Continuar por")
                elif "Turn right" in path:
                    path = path.replace("Turn right", "Girar a la derecha")
                elif "Turn left" in path:
                    path = path.replace("Turn left", "Girar a la izquierda")
                elif "Arrive at destination" in path:
                    path = path.replace("Arrive at destination", "Llegar a destino")
                elif "Keep left" in path:
                    path = path.replace("Keep left", "Mantenerse a la izquierda")
                elif "Keep right" in path:
                    path = path.replace("Keep right", "Mantenerse a la derecha")
                elif "Roundabout" in path:
                    path = path.replace("Roundabout", "Rotonda")
                elif "Enter roundabout" in path:
                    path = path.replace("Enter roundabout", "Entrar a rotonda")
                
                print(f"{path} ({distance/1000:.2f} km / {distance/1000/1.61:.2f} millas)")
            
            print("="*50)
        else:
            print("Error: " + paths_data["message"])
            print("*"*50)
