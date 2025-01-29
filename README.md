import phonenumbers
from phonenumbers import geocoder, carrier, timezone

def track_phone_number(phone_number):
    """
    Melacak informasi nomor telepon termasuk negara, operator, dan zona waktu
    
    Parameters:
    phone_number (str): Nomor telepon dalam format internasional (contoh: +62822297260740)
    
    Returns:
    dict: Dictionary berisi informasi nomor telepon
    """
    try:
        # Parse nomor telepon
        parsed_number = phonenumbers.parse(phone_number)
        
        # Validasi nomor telepon
        if not phonenumbers.is_valid_number(parsed_number):
            return {"error": "Nomor telepon tidak valid"}
            
        # Dapatkan informasi
        info = {
            "nomor": phonenumbers.format_number(parsed_number, phonenumbers.PhoneNumberFormat.INTERNATIONAL),
            "negara": geocoder.description_for_number(parsed_number, "id"),  # 'id' untuk bahasa Indonesia
            "operator": carrier.name_for_number(parsed_number, "id"),
            "zona_waktu": timezone.time_zones_for_number(parsed_number),
            "valid": phonenumbers.is_valid_number(parsed_number),
            "tipe_nomor": "Mobile" if phonenumbers.number_type(parsed_number) == phonenumbers.PhoneNumberType.MOBILE else "Fixed Line"
        }
        
        return info
        
    except Exception as e:
        return {"error": f"Terjadi kesalahan: {str(e)}"}

# Penggunaan dengan nomor yang diberikan
if __name__ == "__main__":
    nomor = "+62822297260740"  # Nomor yang diberikan
    print(f"\nMelacak nomor: {nomor}")
    result = track_phone_number(nomor)
    
    if "error" in result:
        print(f"Error: {result['error']}")
    else:
        print("Informasi nomor:")
        for key, value in result.items():
            print(f"{key}: {value}")
