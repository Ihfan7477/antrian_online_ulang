mport pyodbc
import requests
from datetime import datetime

def post_antrean_ulang():
    API_ENDPOINT = 'http://192.168.10.3:8001//api/saveBPJSANTREANJKN'

    # Database 
    conn_str = (
        "DRIVER={ODBC Driver 17 for SQL Server};"
        "SERVER=192.168.10.2;"
        "DATABASE=TOMBOL_ANTRIAN;"
        "UID=sa;"
        "PWD=RSPUBG567#;"
    )

    # Connect DB
    conn = pyodbc.connect(conn_str)
    cursor = conn.cursor()

    # Tanggal Sekarang
    current_date = datetime.now().date()

    # Ambil data
    cursor.execute("""
        SELECT
            a.*,
            b.Pendaftaran
        FROM
            BPJS_ANTREAN a,
            poliklinik_antri b
        WHERE
            a.bridging_pesan IS NOT NULL 
            AND a.bridging_pesan <> 'data nomorkartu  belum sesuai.' 
            AND a.tanggalperiksa = ?
            AND b.NO = a.nomorantrean 
            AND b.Tanggal = a.tanggalperiksa
    """, current_date)

    datas = cursor.fetchall()

    if not datas:
        print('No records found with the specified conditions.')
        return

    for data in datas:
        kode_booking = data.kodebooking
        parts = kode_booking.split('-')

        # Extract tanggal_antrian and kode_poli
        tanggal_antrian = parts[0] + '-' + parts[1] + '-' + parts[2]  # '2024-09-19'
        kodebooking = parts[3] # PSD001
        kode_poli = parts[4]  # 'PK007'

        # Update
        new_kodebooking = f"{kode_booking}_"
        cursor.execute("UPDATE BPJS_ANTREAN SET kodebooking = ? WHERE kodebooking = ?", new_kodebooking, kode_booking)
        conn.commit()

        # Data Di Kirim
        data = {
            'no_booking': kodebooking,
            'kode_pasien': data.norm,
            'kode_poli': kode_poli,
            'kode_pendaftaran': data.Pendaftaran,
            'no_referensi': data.nomorreferensi,
            'jenis_kunjungan': 2,
        }
        
        try:
            response = requests.get(API_ENDPOINT, params=data)

            # Delete 
            cursor.execute("DELETE FROM BPJS_ANTREAN WHERE kodebooking = ?", new_kodebooking)
            conn.commit()

            print(f'Response {response.status_code}: {response.text}')

            if response.status_code == 200:
                print(f'Bridging Ulang Berhasil for kodebooking: {kode_booking}')
            else:
                print(f'Error for kodebooking {kode_booking}: {response.text}')
        except Exception as e:
            print(f'An error occurred for kodebooking {kode_booking}: {e}')

    cursor.close()
    conn.close()

if _name_ == '_main_':
    post_antrean_ulang()
