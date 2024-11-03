DELIMITER //

CREATE PROCEDURE RegisterUser(
    IN p_name VARCHAR(100),
    IN p_email VARCHAR(100),
    IN p_phone_number VARCHAR(15),
    IN p_password VARCHAR(255),
    IN p_confirm_password VARCHAR(255)
)
BEGIN
    DECLARE email_valid BOOLEAN DEFAULT FALSE;
    DECLARE password_match BOOLEAN DEFAULT FALSE;

    -- Bước 1: Kiểm tra định dạng email
    IF p_email LIKE '%_@__%.__%' THEN
        SET email_valid = TRUE;
    END IF;

    -- Bước 2: Kiểm tra mật khẩu
    IF LENGTH(p_password) >= 8 THEN
        SET password_match = TRUE;
    END IF;

    -- Bước 3: So sánh mật khẩu
    IF p_password = p_confirm_password THEN
        -- Bước 4: Lưu thông tin người dùng nếu tất cả các kiểm tra hợp lệ
        IF email_valid AND password_match THEN
            INSERT INTO Users (name, email, phone_number, password)
            VALUES (p_name, p_email, p_phone_number, SHA2(p_password, 256));
            
            -- Ghi nhận đăng ký thành công
            INSERT INTO RegistrationAttempts (email, success)
            VALUES (p_email, TRUE);
        ELSE
            -- Ghi nhận đăng ký thất bại
            INSERT INTO RegistrationAttempts (email, success)
            VALUES (p_email, FALSE);
            SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Đăng ký không thành công do lỗi xác thực.';
        END IF;
    ELSE
        -- Ghi nhận đăng ký thất bại
        INSERT INTO RegistrationAttempts (email, success)
        VALUES (p_email, FALSE);
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Mật khẩu không khớp.';
    END IF;

END //

DELIMITER ;
