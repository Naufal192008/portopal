-- ============================================================
-- DATABASE: sistem_lelang
-- Aplikasi Lelang Online - FR.IA.02 TPD
-- ============================================================

USE sistem_lelang;

-- ============================================================
-- 1. TABEL LEVEL
-- ============================================================
DROP TABLE IF EXISTS tb_history_lelang;
DROP TABLE IF EXISTS tb_lelang;
DROP TABLE IF EXISTS tb_barang;
DROP TABLE IF EXISTS tb_masyarakat;
DROP TABLE IF EXISTS tb_petugas;
DROP TABLE IF EXISTS tb_level;

CREATE TABLE tb_level (
    id_level INT AUTO_INCREMENT PRIMARY KEY,
    level ENUM('administrator', 'petugas') NOT NULL
);

-- ============================================================
-- 2. TABEL PETUGAS
-- ============================================================
CREATE TABLE tb_petugas (
    id_petugas INT AUTO_INCREMENT PRIMARY KEY,
    nama_petugas VARCHAR(25) NOT NULL,
    username VARCHAR(25) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    id_level INT,
    FOREIGN KEY (id_level) REFERENCES tb_level(id_level) ON DELETE SET NULL
);

-- ============================================================
-- 3. TABEL MASYARAKAT
-- ============================================================
CREATE TABLE tb_masyarakat (
    id_user INT AUTO_INCREMENT PRIMARY KEY,
    nama_lengkap VARCHAR(25) NOT NULL,
    username VARCHAR(25) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    telp VARCHAR(25)
);

-- ============================================================
-- 4. TABEL BARANG
-- ============================================================
CREATE TABLE tb_barang (
    id_barang INT AUTO_INCREMENT PRIMARY KEY,
    nama_barang VARCHAR(25) NOT NULL,
    harga_awal INT NOT NULL,
    deskripsi_barang VARCHAR(100),
    tgl DATE
);

-- ============================================================
-- 5. TABEL LELANG
-- ============================================================
CREATE TABLE tb_lelang (
    id_lelang INT AUTO_INCREMENT PRIMARY KEY,
    id_barang INT NOT NULL,
    tgl_lelang DATE,
    harga_akhir INT,
    id_user INT,
    id_petugas INT,
    status ENUM('dibuka', 'ditutup') DEFAULT 'ditutup',
    FOREIGN KEY (id_barang) REFERENCES tb_barang(id_barang) ON DELETE CASCADE,
    FOREIGN KEY (id_user) REFERENCES tb_masyarakat(id_user) ON DELETE SET NULL,
    FOREIGN KEY (id_petugas) REFERENCES tb_petugas(id_petugas) ON DELETE SET NULL
);

-- ============================================================
-- 6. TABEL HISTORY LELANG
-- ============================================================
CREATE TABLE tb_history_lelang (
    id_history INT AUTO_INCREMENT PRIMARY KEY,
    id_lelang INT,
    id_barang INT,
    id_user INT,
    penawaran_harga INT,
    FOREIGN KEY (id_lelang) REFERENCES tb_lelang(id_lelang) ON DELETE CASCADE,
    FOREIGN KEY (id_barang) REFERENCES tb_barang(id_barang) ON DELETE CASCADE,
    FOREIGN KEY (id_user) REFERENCES tb_masyarakat(id_user) ON DELETE CASCADE
);

-- ============================================================
-- DATA AWAL (SEEDER)
-- ============================================================

-- Level
INSERT INTO tb_level (level) VALUES ('administrator'), ('petugas');

-- Petugas (password: admin123, petugas123)
-- Hash pakai password_hash() PHP, default password di bawah
INSERT INTO tb_petugas (nama_petugas, username, password, id_level) VALUES
('Administrator', 'admin', '$2y$10$eImiTXuWVxfM37uY4JANjQ==eIWH7CbvBk8eLZXzZ0P0eIWH7CbvBk8eLZXzZ0P0', 1),
('Petugas Lelang', 'petugas1', '$2y$10$eImiTXuWVxfM37uY4JANjQ==eIWH7CbvBk8eLZXzZ0P0eIWH7CbvBk8eLZXzZ0P0', 2);

-- Masyarakat
INSERT INTO tb_masyarakat (nama_lengkap, username, password, telp) VALUES
('Budi Santoso', 'budi', '$2y$10$eImiTXuWVxfM37uY4JANjQ==eIWH7CbvBk8eLZXzZ0P0eIWH7CbvBk8eLZXzZ0P0', '081234567890'),
('Ani Wijaya', 'ani', '$2y$10$eImiTXuWVxfM37uY4JANjQ==eIWH7CbvBk8eLZXzZ0P0eIWH7CbvBk8eLZXzZ0P0', '089876543210');

-- Barang
INSERT INTO tb_barang (nama_barang, harga_awal, deskripsi_barang, tgl) VALUES
('Sepeda Motor', 5000000, 'Honda Beat 2020', CURDATE()),
('Laptop Asus', 3000000, 'Asus Vivobook 2019', CURDATE()),
('Handphone', 1500000, 'Samsung A50', CURDATE());

-- ============================================================
-- STORED PROCEDURE
-- ============================================================

DELIMITER //

-- SP 1: Buka Lelang
DROP PROCEDURE IF EXISTS sp_buka_lelang //
CREATE PROCEDURE sp_buka_lelang(IN p_id_lelang INT)
BEGIN
    UPDATE tb_lelang SET status = 'dibuka' WHERE id_lelang = p_id_lelang;
END //

-- SP 2: Tutup Lelang
DROP PROCEDURE IF EXISTS sp_tutup_lelang //
CREATE PROCEDURE sp_tutup_lelang(IN p_id_lelang INT)
BEGIN
    DECLARE v_harga_akhir INT;
    DECLARE v_id_user INT;
    
    -- Ambil penawaran tertinggi
    SELECT MAX(penawaran_harga), id_user INTO v_harga_akhir, v_id_user
    FROM tb_history_lelang 
    WHERE id_lelang = p_id_lelang
    GROUP BY id_user
    ORDER BY penawaran_harga DESC
    LIMIT 1;
    
    -- Update lelang
    UPDATE tb_lelang 
    SET status = 'ditutup', harga_akhir = v_harga_akhir, id_user = v_id_user
    WHERE id_lelang = p_id_lelang;
END //

-- SP 3: Tambah Penawaran
DROP PROCEDURE IF EXISTS sp_tambah_penawaran //
CREATE PROCEDURE sp_tambah_penawaran(
    IN p_id_lelang INT,
    IN p_id_barang INT,
    IN p_id_user INT,
    IN p_penawaran INT
)
BEGIN
    DECLARE v_harga_tertinggi INT;
    DECLARE v_status VARCHAR(10);
    
    -- Cek status lelang
    SELECT status INTO v_status FROM tb_lelang WHERE id_lelang = p_id_lelang;
    
    IF v_status = 'ditutup' THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Lelang sudah ditutup!';
    END IF;
    
    -- Cek penawaran tertinggi saat ini
    SELECT MAX(penawaran_harga) INTO v_harga_tertinggi
    FROM tb_history_lelang WHERE id_lelang = p_id_lelang;
    
    IF v_harga_tertinggi IS NULL OR p_penawaran > v_harga_tertinggi THEN
        INSERT INTO tb_history_lelang (id_lelang, id_barang, id_user, penawaran_harga)
        VALUES (p_id_lelang, p_id_barang, p_id_user, p_penawaran);
    ELSE
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Penawaran harus lebih tinggi dari sebelumnya!';
    END IF;
END //

-- SP 4: Laporan Lelang
DROP PROCEDURE IF EXISTS sp_laporan_lelang //
CREATE PROCEDURE sp_laporan_lelang(IN p_start DATE, IN p_end DATE)
BEGIN
    SELECT 
        l.id_lelang,
        b.nama_barang,
        b.harga_awal,
        l.harga_akhir,
        l.tgl_lelang,
        l.status,
        m.nama_lengkap AS pemenang,
        p.nama_petugas
    FROM tb_lelang l
    JOIN tb_barang b ON l.id_barang = b.id_barang
    LEFT JOIN tb_masyarakat m ON l.id_user = m.id_user
    LEFT JOIN tb_petugas p ON l.id_petugas = p.id_petugas
    WHERE l.tgl_lelang BETWEEN p_start AND p_end
    ORDER BY l.tgl_lelang DESC;
END //

DELIMITER ;

-- ============================================================
-- VERIFIKASI
-- ============================================================
SELECT '=== LEVEL ===' AS '';
SELECT * FROM tb_level;

SELECT '=== PETUGAS ===' AS '';
SELECT id_petugas, nama_petugas, username, id_level FROM tb_petugas;

SELECT '=== MASYARAKAT ===' AS '';
SELECT id_user, nama_lengkap, username, telp FROM tb_masyarakat;

SELECT '=== BARANG ===' AS '';
SELECT * FROM tb_barang;

SELECT '=== STORED PROCEDURES ===' AS '';
SHOW PROCEDURE STATUS WHERE Db = 'sistem_lelang';