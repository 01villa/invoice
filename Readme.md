### Crear un función y un trigger para validar que el numero de cedula del cliente tenga 10 números (no letras) en la tabla cliente.

## Funcion
```sql
CREATE OR REPLACE FUNCTION validar_cedula(cedula TEXT) 
RETURNS BOOLEAN AS $$
BEGIN
    IF cedula ~ '^\d{10}$' THEN
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END IF;
END;
$$ LANGUAGE plpgsql;
```
## Trigger
```sql
CREATE OR REPLACE FUNCTION validar_cedula_trigger() 
RETURNS TRIGGER AS $$
BEGIN
    IF NOT validar_cedula(NEW.cedula) THEN
        RAISE EXCEPTION 'Número de cédula inválido. Debe contener exactamente 10 dígitos.';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```


### Crear un función y un trigger para que cada vez que se inserte un nuevo registro en la tabla item se disminuya el stock de la tabla product.

## Funcion
```sql
CREATE OR REPLACE FUNCTION disminuir_stock() 
RETURNS TRIGGER AS $$
BEGIN
    UPDATE product
    SET stock = stock - NEW.cantidad
    WHERE id = NEW.product_id;

    IF (SELECT stock FROM product WHERE id = NEW.product_id) < 0 THEN
        RAISE EXCEPTION 'Stock insuficiente para el producto.';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

## Trigger
```sql
CREATE TRIGGER trg_disminuir_stock
AFTER INSERT ON item
FOR EACH ROW EXECUTE PROCEDURE disminuir_stock();

```
### Crear un función y un trigger para la tabla invoice donde valide que el campo create_at sea del año actual (fecha sistema).

## Funcion
 
```sql
CREATE OR REPLACE FUNCTION validar_fecha_actual() 
RETURNS TRIGGER AS $$
BEGIN
    IF EXTRACT(YEAR FROM NEW.create_at) != EXTRACT(YEAR FROM CURRENT_DATE) THEN
        RAISE EXCEPTION 'La fecha de la factura debe ser del año actual.';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

## Trigger
```sql
CREATE TRIGGER trg_validar_fecha_actual
BEFORE INSERT OR UPDATE ON invoice
FOR EACH ROW EXECUTE PROCEDURE validar_fecha_actual();
```

### Crear un función y un trigger para la tabla client y validar que el correo tenga un @.

## Funcion
```sql
CREATE OR REPLACE FUNCTION validar_correo() 
RETURNS TRIGGER AS $$
BEGIN
    IF POSITION('@' IN NEW.email) = 0 THEN
        RAISE EXCEPTION 'El correo debe contener un @.';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

## Trigger
```sql
CREATE TRIGGER trg_validar_correo
BEFORE INSERT OR UPDATE ON cliente
FOR EACH ROW EXECUTE PROCEDURE validar_correo();
```