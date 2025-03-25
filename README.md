from PIL import Image
import numpy as np

def encode_image(image_path, message, output_path):
    # Load the image
    image = Image.open(image_path)
    image = image.convert('RGB')
    pixels = np.array(image)

    # Convert the message to binary
    binary_message = ''.join(format(ord(char), '08b') for char in message) + '00000000'  # Null character to signify end
    data_index = 0

    # Embed the message into the image
    for row in pixels:
        for pixel in row:
            if data_index < len(binary_message):
                # Change the LSB of the pixel
                pixel[0] = (pixel[0] & ~1) | int(binary_message[data_index])
                data_index += 1
            else:
                break

    # Save the modified image
    modified_image = Image.fromarray(pixels)
    modified_image.save(output_path)
    print(f"Message encoded into {output_path}")

def decode_image(image_path):
    # Load the image
    image = Image.open(image_path)
    pixels = np.array(image)
    binary_message = ""

    # Extract the message from the image
    for row in pixels:
        for pixel in row:
            binary_message += str(pixel[0] & 1)  # Get the LSB
            if binary_message[-8:] == '00000000':  # Check for null character
                break

    # Convert binary to string
    message = ""
    for i in range(0, len(binary_message) - 8, 8):
        byte = binary_message[i:i + 8]
        message += chr(int(byte, 2))

    return message

# Example Usage
image_path = 'input_image.png'  # Path to the input image
output_path = 'output_image.png'  # Path to save the modified image
secret_message = ''

# Read the secret message from a file
with open('secret_message.txt', 'r') as file:
    secret_message = file.read()

# Encode the message into the image
encode_image(image_path, secret_message, output_path)

# Decode the message from the modified image
extracted_message = decode_image(output_path)
print(f"Extracted Message: {extracted_message}")
