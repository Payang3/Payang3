import phonenumbers
from phonenumbers import geocoder, carrier, timezone
import folium
from geopy.geocoders import Nominatim
import time

def lacak_lokasi_indonesia(nomor_telepon):
    """
    Melacak lokasi nomor telepon Indonesia dan menampilkan pada peta
    
    Parameters:
    nomor_telepon (str): Nomor telepon format internasional (+62)
    """
    try:
        # Bersihkan format nomor
        nomor_bersih = nomor_telepon.replace("-", "").replace(" ", "")
        
        # Parse nomor telepon
        parsed_number = phonenumbers.parse(nomor_bersih)
        
        # Validasi nomor Indonesia
        if not (phonenumbers.is_valid_number(parsed_number) and parsed_number.country_code == 62):
            raise ValueError("Nomor telepon harus nomor Indonesia yang valid")
        
        # Dapatkan info lokasi
        lokasi = geocoder.description_for_number(parsed_number, "id")
        operator = carrier.name_for_number(parsed_number, "id")
        zona = timezone.time_zones_for_number(parsed_number)
        
        # Inisialisasi geocoder
        geolocator = Nominatim(user_agent="phone_locator")
        
        # Cari koordinat (fokus ke Indonesia)
        location = geolocator.geocode(f"{lokasi}, Indonesia", exactly_one=True)
        time.sleep(1)  # Delay untuk menghormati rate limit
        
        if location:
            # Buat peta
            peta = folium.Map(location=[location.latitude, location.longitude], 
                            zoom_start=12)
            
            # Tambah marker
            folium.Marker(
                [location.latitude, location.longitude],
                popup=f"""
                Nomor: {nomor_telepon}
                Lokasi: {lokasi}
                Operator: {operator}
                Zona Waktu: {', '.join(zona)}
                Koordinat: {location.latitude}, {location.longitude}
                """,
                icon=folium.Icon(color='red', icon='info-sign')
            ).add_to(peta)
            
            # Simpan peta
            peta_file = "lokasi_nomor_indonesia.html"
            peta.save(peta_file)
            
            return {
                "nomor": nomor_telepon,
                "lokasi": lokasi,
                "operator": operator,
                "zona_waktu": zona,
                "latitude": location.latitude,
                "longitude": location.longitude,
                "file_peta": peta_file
            }
        else:
            raise ValueError("Tidak dapat menemukan koordinat untuk lokasi tersebut")
            
    except Exception as e:
        return {"error": str(e)}

# Jalankan pelacakan untuk nomor yang diberikan
if __name__ == "__main__":
    nomor = "+62822297260740"  # Nomor yang Anda berikan
    
    print("Melacak lokasi...")
    hasil = lacak_lokasi_indonesia(nomor)
    
    if "error" in hasil:
        print(f"Error: {hasil['error']}")
    else:
        print("\n=== Informasi Lokasi ===")
        print(f"Nomor: {hasil['nomor']}")
        print(f"Lokasi: {hasil['lokasi']}")
        print(f"Operator: {hasil['operator']}")
        print(f"Zona Waktu: {', '.join(hasil['zona_waktu'])}")
        print(f"Koordinat: {hasil['latitude']}, {hasil['longitude']}")
        print(f"\nPeta telah disimpan ke: {hasil['file_peta']}")
        print("\nBuka file HTML tersebut di browser untuk melihat lokasi pada peta")
