from PIL import Image

def hide_text_in_image(image_path, text_path, output_path):
    """
    Hides text from a file into an image using LSB steganography.

    Args:
        image_path: Path to the input image.
        text_path: Path to the text file to hide.
        output_path: Path to save the output image.
    """
    try:
        img = Image.open(image_path).convert('RGBA')  # Use RGBA to handle transparency
        width, height = img.size

        with open(text_path, 'rb') as f: # Open in binary mode to handle all characters.
            text_bytes = f.read()

        text_binary = ''.join(format(byte, '08b') for byte in text_bytes)

        if len(text_binary) > width * height * 3:
            raise ValueError("Text too large to hide in image.")

        pixels = list(img.getdata())
        binary_index = 0

        new_pixels = []
        for pixel in pixels:
            r, g, b, a = pixel  # Keep alpha for RGBA
            if binary_index < len(text_binary):
                r = (r & ~1) | int(text_binary[binary_index])
                binary_index += 1
            if binary_index < len(text_binary):
                g = (g & ~1) | int(text_binary[binary_index])
                binary_index += 1
            if binary_index < len(text_binary):
                b = (b & ~1) | int(text_binary[binary_index])
                binary_index += 1
            new_pixels.append((r, g, b, a)) # include Alpha

        img.putdata(new_pixels)
        img.save(output_path)
        print(f"Text hidden successfully! Saved to {output_path}")

    except FileNotFoundError:
        print("Error: Image or text file not found.")
    except ValueError as e:
        print(f"Error: {e}")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")

def extract_text_from_image(image_path, output_text_path):
    """
    Extracts hidden text from an image.

    Args:
        image_path: Path to the image containing hidden text.
        output_text_path: path to save the extracted text.
    """
    try:
        img = Image.open(image_path).convert('RGBA')
        pixels = list(img.getdata())

        binary_data = ''
        for pixel in pixels:
            r, g, b, a = pixel #include Alpha
            binary_data += str(r & 1)
            binary_data += str(g & 1)
            binary_data += str(b & 1)

        all_bytes = [binary_data[i: i + 8] for i in range(0, len(binary_data), 8)]

        decoded_bytes = bytearray()
        for byte in all_bytes:

            if len(byte) == 8: #prevent errors from partial bytes at the end.
                try:
                  decoded_bytes.append(int(byte, 2))
                except ValueError:
                  break #handle possible corruption at the end of the image.

        with open(output_text_path, 'wb') as f:
            f.write(decoded_bytes)

        print(f"Text extracted successfully! Saved to {output_text_path}")

    except FileNotFoundError:
        print("Error: Image file not found.")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")

# Example usage:
image_path = "pic.png"
text_path = "hidden.txt"
output_image_path = "pic_hidden.png"
extracted_text_path = "extracted_hidden.txt"

hide_text_in_image(image_path, text_path, output_image_path)
extract_text_from_image(output_image_path, extracted_text_path)
