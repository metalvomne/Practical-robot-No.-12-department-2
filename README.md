from PIL import Image, ImageDraw, ImageFont
import os

def get_user_input():
    """Отримання параметрів від користувача"""
    image_path = input("Введіть шлях до зображення (наприклад, background.jpg): ")
    text = input("Введіть текст для листівки: ")
    color = input("Введіть колір тексту (наприклад, 'red', 'blue', '#00FF00' або українською): ")
    position = input("Введіть позицію тексту (центр, верх, низ): ").lower()
    return image_path, text, color, position


def add_text_to_image(image_path, text, color, position):
    """Додавання тексту до зображення"""
    try:
        # Відкриваємо зображення
        image = Image.open(image_path)
        draw = ImageDraw.Draw(image)

        # Мапа українських назв кольорів
        colors = {
            "синій": "blue", "червоний": "red", "зелений": "green",
            "білий": "white", "чорний": "black", "жовтий": "yellow",
            "рожевий": "pink", "оранжевий": "orange", "фіолетовий": "purple"
        }
        color = colors.get(color.lower(), color)

        # Вибір шрифту
        try:
            font = ImageFont.truetype("arial.ttf", size=60)
        except:
            font = ImageFont.load_default()

        img_width, img_height = image.size

        # Якщо текст занадто довгий — автоматично зменшуємо шрифт
        max_width = int(img_width * 0.9)  # не більше 90% ширини зображення
        while True:
            bbox = draw.textbbox((0, 0), text, font=font)
            text_width = bbox[2] - bbox[0]
            if text_width <= max_width or font.size <= 20:
                break
            font = ImageFont.truetype("arial.ttf", size=font.size - 2)

        # Розбиваємо довгий текст на рядки
        words = text.split()
        lines = []
        current_line = words[0]
        for word in words[1:]:
            test_line = current_line + ' ' + word
            test_bbox = draw.textbbox((0, 0), test_line, font=font)
            if test_bbox[2] - test_bbox[0] <= max_width:
                current_line = test_line
            else:
                lines.append(current_line)
                current_line = word
        lines.append(current_line)

        # Розрахунок координат для розміщення блоку тексту
        line_height = bbox[3] - bbox[1] + 10
        total_text_height = len(lines) * line_height

        if position == "центр":
            y = (img_height - total_text_height) / 2
        elif position == "верх":
            y = 50
        elif position == "низ":
            y = img_height - total_text_height - 50
        else:
            print("Невідома позиція. Використано 'центр'.")
            y = (img_height - total_text_height) / 2

        # Малюємо кожен рядок тексту
        for line in lines:
            line_bbox = draw.textbbox((0, 0), line, font=font)
            line_width = line_bbox[2] - line_bbox[0]
            x = (img_width - line_width) / 2
            draw.text((x, y), line, fill=color, font=font)
            y += line_height

        # Збереження результату
        output_path = os.path.join(os.path.dirname(image_path), "листівка.jpg")
        image.save(output_path)
        print(f"Листівка успішно створена: {output_path}")

    except FileNotFoundError:
        print("Помилка: файл не знайдено. Перевірте шлях до зображення.")
    except OSError:
        print("Помилка: не вдалося відкрити файл. Можливо, це не зображення.")
    except Exception as e:
        print(f"Виникла помилка: {e}")


def main():
    """Основна функція"""
    print("=== Створення персоналізованої листівки ===")
    image_path, text, color, position = get_user_input()
    add_text_to_image(image_path, text, color, position)


if __name__ == "__main__":
    main()
