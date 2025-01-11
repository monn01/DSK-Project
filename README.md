# DSK-Project
Tugas Akhir_DSKProject_WordCOunter_2400018119_Muhammad Dzaky Prayata_C
; Program Penghitung Kata (Word Counter)  (Revisi 4>

.model small
.stack 100h

.data
    welcome_msg db 'Selamat datang di program Word Counter!', 0Dh, 0Ah, '$'
    prompt db 0Dh, 0Ah, 'Masukkan teks: $'
    output_word_count db 0Dh, 0Ah, 'Jumlah kata: $'
    output_char_count db 0Dh, 0Ah, 'Jumlah karakter (dengan spasi): $'
    output_char_no_space db 0Dh, 0Ah, 'Jumlah karakter (tanpa spasi): $'
    retry_msg db 0Dh, 0Ah, 'Apakah kamu ingin menghitung teks lagi? (Y/N): $'
    invalid_input_msg db 0Dh, 0Ah, 'Input tidak valid, coba lagi.$'
    bye_msg db 0Dh, 0Ah, 'Terima kasih telah menggunakan program ini!', 0Dh, 0Ah, '$'
    input_buffer db 255, 0, 255 dup('$') ; Buffer input (255 karakter maksimum)
    word_count dw 0             ; Penyimpanan sementara jumlah kata
    char_count dw 0             ; Penyimpanan sementara jumlah karakter dengan spasi
    char_no_space dw 0          ; Penyimpanan sementara jumlah karakter tanpa spasi

.code
main proc
    ; Inisialisasi segmen data
    mov ax, @data
    mov ds, ax

    ; Tampilkan pesan selamat datang
    lea dx, welcome_msg
    mov ah, 09h
    int 21h

    ; Label untuk input ulang
input_loop:
    ; Reset penghitung kata dan karakter
    mov word_count, 0
    mov char_count, 0
    mov char_no_space, 0

    ; Tampilkan prompt untuk input teks
    lea dx, prompt
    mov ah, 09h
    int 21h

    ; Ambil input teks dari pengguna
    lea dx, input_buffer
    mov ah, 0Ah
    int 21h

    ; Hitung jumlah kata dan karakter
    mov si, offset input_buffer + 2 ; Pointer ke awal teks (lewati panjang input byte pertama)
    mov bl, 0 ; 0 = di luar kata, 1 = di dalam kata

process_input:
    lodsb                      ; Ambil karakter berikutnya
    cmp al, 0Dh                ; Akhir input (Enter)
    je finalize_counts

    inc char_count             ; Tambahkan jumlah karakter total
    cmp al, ' '                ; Cek apakah spasi
    je outside_word

    ; Jika bukan spasi, tambahkan jumlah karakter tanpa spasi
    inc char_no_space

    ; Jika sebelumnya di luar kata, maka ini awal kata baru
    cmp bl, 1
    je process_input
    mov bl, 1
    inc word_count
    jmp process_input

outside_word:
    mov bl, 0                  ; Set flag ke luar kata
    jmp process_input

finalize_counts:
    ; Tampilkan jumlah kata
    lea dx, output_word_count
    mov ah, 09h
    int 21h
    mov ax, word_count
    call print_number

    ; Tampilkan jumlah karakter dengan spasi
    lea dx, output_char_count
    mov ah, 09h
    int 21h
    mov ax, char_count
    call print_number

    ; Tampilkan jumlah karakter tanpa spasi
    lea dx, output_char_no_space
    mov ah, 09h
    int 21h
    mov ax, char_no_space
    call print_number

    ; Tanya apakah ingin mengulang
retry_input:
    lea dx, retry_msg
    mov ah, 09h
    int 21h
    mov ah, 01h
    int 21h
    cmp al, 'Y'
    je input_loop
    cmp al, 'y'
    je input_loop
    cmp al, 'N'
    je exit_program
    cmp al, 'n'
    je exit_program

    ; Tampilkan pesan input tidak valid
    lea dx, invalid_input_msg
    mov ah, 09h
    int 21h
    jmp retry_input

exit_program:
    ; Tampilkan pesan selamat tinggal
    lea dx, bye_msg
    mov ah, 09h
    int 21h
    ret

print_number proc
    ; Subroutine untuk menampilkan angka dari register AX
    push dx
    push cx
    push bx

    xor cx, cx          ; Reset digit counter
    mov bx, 10

convert_digits:
    xor dx, dx
    div bx               ; Bagi angka, simpan sisa di DX
    push dx              ; Simpan sisa (digit saat ini)
    inc cx               ; Tambahkan penghitung digit
    test ax, ax
    jnz convert_digits

print_digits:
    pop dx               ; Ambil digit terakhir
    add dl, '0'          ; Konversi ke ASCII
    mov ah, 02h
    int 21h              ; Cetak digit
    loop print_digits

    pop bx
    pop cx
    pop dx
    ret
print_number endp

end main
