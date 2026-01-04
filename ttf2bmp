# ==========================================
# TTF2BMP - Font .ttf to BMP Converter
#
# Author: Nika Falaleeva
# GitHub: https://github.com/nikanikoo
#
# Description:
# A tool to convert TrueType Font (.ttf) files into 
# bitmap images (.bmp) with customizable parameters.
#
# License: MIT
# ==========================================

import sys
import argparse
from pathlib import Path
from PIL import Image, ImageDraw, ImageFont
from PyQt6.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, 
                             QHBoxLayout, QPushButton, QLabel, QLineEdit, 
                             QSpinBox, QFileDialog, QTextEdit, QGroupBox,
                             QColorDialog, QCheckBox)
from PyQt6.QtCore import Qt, QThread, pyqtSignal
from PyQt6.QtGui import QFont as QGuiFont


class FontConverterCore:
    """Core class for font to BMP conversion"""
    
    def __init__(self):
        self.reset_defaults()
    
    def reset_defaults(self):
        self.fontpath = "arial.ttf"
        self.fontsize = 16
        self.char_width = 16
        self.char_height = 16
        self.chars_per_row = 16
        self.output_path = "output.bmp"
        self.bg_color = (255, 192, 203)
        self.text_color = (255, 255, 255)
        self.outline_color = (0, 0, 0)
        self.use_outline = False
        self.draw_grid = False
        
    def get_character_set(self):
        """Generate full character set"""
        ascii_chars = ''.join(chr(i) for i in range(32, 127))
        chars_upper = 'ABVGDEZHZIJKLMNOPRSTUFHCCHSHSHYEYUYA'
        chars_lower = 'abvgdezhzijklmnoprstufhcchshshyeyuya'
        extra_symbols = '№§©®™°±×÷'
        return ascii_chars + chars_upper + chars_lower + extra_symbols
    
    def draw_char_with_outline(self, image, draw, pos, char, font):
        """Draw character with optional outline"""
        x, y = pos
        
        temp = Image.new('RGBA', (self.char_width * 2, self.char_height * 2), (0, 0, 0, 0))
        temp_draw = ImageDraw.Draw(temp)
        
        bbox = font.getbbox(char)
        char_w = bbox[2] - bbox[0]
        char_h = bbox[3] - bbox[1]
        
        offset_x = (self.char_width - char_w) // 2 - bbox[0]
        offset_y = (self.char_height - char_h) // 2 - bbox[1]
        
        if self.use_outline:
            for dx in [-1, 0, 1]:
                for dy in [-1, 0, 1]:
                    if dx == 0 and dy == 0:
                        continue
                    temp_draw.text((offset_x + dx, offset_y + dy), char, 
                                 font=font, fill=self.outline_color)
        
        temp_draw.text((offset_x, offset_y), char, font=font, fill=self.text_color)
        
        temp = temp.crop((0, 0, self.char_width, self.char_height))
        temp_rgb = Image.new('RGB', (self.char_width, self.char_height), self.bg_color)
        temp_rgb.paste(temp, (0, 0), temp)
        image.paste(temp_rgb, (x, y))
    
    def convert(self, progress_callback=None):
        """Main conversion function"""
        all_chars = self.get_character_set()
        total_chars = len(all_chars)
        rows = (total_chars + self.chars_per_row - 1) // self.chars_per_row
        img_width = self.char_width * self.chars_per_row
        img_height = self.char_height * rows
        
        image = Image.new('RGB', (img_width, img_height), self.bg_color)
        draw = ImageDraw.Draw(image)
        
        try:
            font = ImageFont.truetype(self.fontpath, self.fontsize)
        except Exception as e:
            raise Exception(f"Font loading error: {e}")
        
        for i, char in enumerate(all_chars):
            row = i // self.chars_per_row
            col = i % self.chars_per_row
            
            x = col * self.char_width
            y = row * self.char_height
            
            self.draw_char_with_outline(image, draw, (x, y), char, font)
            
            if self.draw_grid:
                draw.rectangle([x, y, x + self.char_width - 1, y + self.char_height - 1], 
                             outline=(128, 128, 128))
            
            if progress_callback:
                progress_callback(int((i + 1) / total_chars * 100))
        
        image_16color = image.convert('P', palette=Image.ADAPTIVE, colors=16)
        image_16color.save(self.output_path, bits=16)
        
        return {
            'total_chars': total_chars,
            'rows': rows,
            'width': img_width,
            'height': img_height,
            'output': self.output_path
        }


class ConversionThread(QThread):
    """Thread for conversion without blocking GUI"""
    progress = pyqtSignal(int)
    finished = pyqtSignal(dict)
    error = pyqtSignal(str)
    
    def __init__(self, converter):
        super().__init__()
        self.converter = converter
    
    def run(self):
        try:
            result = self.converter.convert(self.progress.emit)
            self.finished.emit(result)
        except Exception as e:
            self.error.emit(str(e))


class FontConverterGUI(QMainWindow):
    """GUI interface"""
    
    def __init__(self):
        super().__init__()
        self.converter = FontConverterCore()
        self.thread = None
        self.init_ui()
    
    def init_ui(self):
        self.setWindowTitle('TTF2BMP by nikanikoo')
        self.setMinimumSize(600, 700)
        
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        layout = QVBoxLayout(central_widget)
        
        # Font parameters group
        font_group = QGroupBox("Font Parameters")
        font_layout = QVBoxLayout()
        
        # Font path
        font_path_layout = QHBoxLayout()
        font_path_layout.addWidget(QLabel("Font:"))
        self.font_path_edit = QLineEdit(self.converter.fontpath)
        font_path_layout.addWidget(self.font_path_edit)
        self.browse_btn = QPushButton("Browse...")
        self.browse_btn.clicked.connect(self.browse_font)
        font_path_layout.addWidget(self.browse_btn)
        font_layout.addLayout(font_path_layout)
        
        # Font size
        size_layout = QHBoxLayout()
        size_layout.addWidget(QLabel("Font size:"))
        self.fontsize_spin = QSpinBox()
        self.fontsize_spin.setRange(8, 72)
        self.fontsize_spin.setValue(self.converter.fontsize)
        size_layout.addWidget(self.fontsize_spin)
        size_layout.addStretch()
        font_layout.addLayout(size_layout)
        
        font_group.setLayout(font_layout)
        layout.addWidget(font_group)
        
        # Character dimensions group
        char_group = QGroupBox("Character Dimensions")
        char_layout = QVBoxLayout()
        
        # Character width and height
        char_size_layout = QHBoxLayout()
        char_size_layout.addWidget(QLabel("Width:"))
        self.char_width_spin = QSpinBox()
        self.char_width_spin.setRange(8, 64)
        self.char_width_spin.setValue(self.converter.char_width)
        char_size_layout.addWidget(self.char_width_spin)
        
        char_size_layout.addWidget(QLabel("Height:"))
        self.char_height_spin = QSpinBox()
        self.char_height_spin.setRange(8, 64)
        self.char_height_spin.setValue(self.converter.char_height)
        char_size_layout.addWidget(self.char_height_spin)
        char_layout.addLayout(char_size_layout)
        
        # Characters per row
        row_layout = QHBoxLayout()
        row_layout.addWidget(QLabel("Characters per row"))
        self.chars_per_row_spin = QSpinBox()
        self.chars_per_row_spin.setRange(8, 32)
        self.chars_per_row_spin.setValue(self.converter.chars_per_row)
        row_layout.addWidget(self.chars_per_row_spin)
        row_layout.addStretch()
        char_layout.addLayout(row_layout)
        
        char_group.setLayout(char_layout)
        layout.addWidget(char_group)
        
        # Colors and options group
        color_group = QGroupBox("Colors and Options")
        color_layout = QVBoxLayout()
        
        # Background color
        bg_layout = QHBoxLayout()
        bg_layout.addWidget(QLabel("Background color:"))
        self.bg_color_btn = QPushButton("Choose")
        self.bg_color_btn.clicked.connect(lambda: self.choose_color('bg'))
        bg_layout.addWidget(self.bg_color_btn)
        self.bg_color_label = QLabel(f"RGB{self.converter.bg_color}")
        bg_layout.addWidget(self.bg_color_label)
        bg_layout.addStretch()
        color_layout.addLayout(bg_layout)
        
        # Text color
        text_layout = QHBoxLayout()
        text_layout.addWidget(QLabel("Text color:"))
        self.text_color_btn = QPushButton("Choose")
        self.text_color_btn.clicked.connect(lambda: self.choose_color('text'))
        text_layout.addWidget(self.text_color_btn)
        self.text_color_label = QLabel(f"RGB{self.converter.text_color}")
        text_layout.addWidget(self.text_color_label)
        text_layout.addStretch()
        color_layout.addLayout(text_layout)
        
        # Checkboxes
        self.outline_check = QCheckBox("Use outline")
        self.outline_check.setChecked(self.converter.use_outline)
        color_layout.addWidget(self.outline_check)
        
        self.grid_check = QCheckBox("Draw grid")
        self.grid_check.setChecked(self.converter.draw_grid)
        color_layout.addWidget(self.grid_check)
        
        color_group.setLayout(color_layout)
        layout.addWidget(color_group)
        
        # Output File
        output_layout = QHBoxLayout()
        output_layout.addWidget(QLabel("Output file:"))
        self.output_edit = QLineEdit(self.converter.output_path)
        output_layout.addWidget(self.output_edit)
        self.save_as_btn = QPushButton("Save as...")
        self.save_as_btn.clicked.connect(self.save_as)
        output_layout.addWidget(self.save_as_btn)
        layout.addLayout(output_layout)
        
        # Convert button
        self.convert_btn = QPushButton("Convert")
        self.convert_btn.setFont(QGuiFont("Arial", 12, QGuiFont.Weight.Bold))
        self.convert_btn.setMinimumHeight(40)
        self.convert_btn.clicked.connect(self.start_conversion)
        layout.addWidget(self.convert_btn)
        
        # Logs
        self.log_text = QTextEdit()
        self.log_text.setReadOnly(True)
        self.log_text.setMaximumHeight(150)
        layout.addWidget(QLabel("Logs:"))
        layout.addWidget(self.log_text)

        # Author link (bottom right)
        author_layout = QHBoxLayout()
        author_layout.addStretch()
        author_label = QLabel('<a href="https://github.com/nikanikoo">by nikanikoo</a>')
        author_label.setOpenExternalLinks(True)
        author_label.setStyleSheet("QLabel { color: #888; font-size: 10px; }")
        author_layout.addWidget(author_label)
        layout.addLayout(author_layout)
    
    def browse_font(self):
        filename, _ = QFileDialog.getOpenFileName(
            self, "Select font", "", "Font Files (*.ttf *.otf);;All Files (*)"
        )
        if filename:
            self.font_path_edit.setText(filename)
    
    def save_as(self):
        filename, _ = QFileDialog.getSaveFileName(
            self, "Save as", self.output_edit.text(), "BMP Files (*.bmp)"
        )
        if filename:
            self.output_edit.setText(filename)
    
    def choose_color(self, color_type):
        current_color = (self.converter.bg_color if color_type == 'bg' 
                        else self.converter.text_color)
        
        color = QColorDialog.getColor()
        if color.isValid():
            rgb = (color.red(), color.green(), color.blue())
            if color_type == 'bg':
                self.converter.bg_color = rgb
                self.bg_color_label.setText(f"RGB{rgb}")
            else:
                self.converter.text_color = rgb
                self.text_color_label.setText(f"RGB{rgb}")
    
    def log(self, message):
        self.log_text.append(message)
        self.log_text.verticalScrollBar().setValue(
            self.log_text.verticalScrollBar().maximum()
        )
    
    def start_conversion(self):
        # Update converter parameters
        self.converter.fontpath = self.font_path_edit.text()
        self.converter.fontsize = self.fontsize_spin.value()
        self.converter.char_width = self.char_width_spin.value()
        self.converter.char_height = self.char_height_spin.value()
        self.converter.chars_per_row = self.chars_per_row_spin.value()
        self.converter.output_path = self.output_edit.text()
        self.converter.use_outline = self.outline_check.isChecked()
        self.converter.draw_grid = self.grid_check.isChecked()
        
        self.log("\n--- Conversion started ---")
        self.convert_btn.setEnabled(False)
        
        self.thread = ConversionThread(self.converter)
        self.thread.progress.connect(self.update_progress)
        self.thread.finished.connect(self.conversion_finished)
        self.thread.error.connect(self.conversion_error)
        self.thread.start()
    
    def update_progress(self, value):
        self.log(f"Progress: {value}%")
    
    def conversion_finished(self, result):
        self.log(f"\n✓ Conversion completed!")
        self.log(f"Characters: {result['total_chars']}")
        self.log(f"Rows: {result['rows']}")
        self.log(f"Size: {result['width']}x{result['height']}")
        self.log(f"File: {result['output']}")
        self.convert_btn.setEnabled(True)
    
    def conversion_error(self, error):
        self.log(f"\n✗ Error: {error}")
        self.convert_btn.setEnabled(True)


def cli_mode():
    """CLI"""
    parser = argparse.ArgumentParser(description='TTF2BMP converter')
    parser.add_argument('-f', '--font', default='ithaca.ttf', help='Path to font file')
    parser.add_argument('-s', '--size', type=int, default=16, help='Font size')
    parser.add_argument('-w', '--width', type=int, default=16, help='Character width')
    parser.add_argument('-H', '--height', type=int, default=16, help='Character height')
    parser.add_argument('-r', '--row', type=int, default=16, help='Characters per row')
    parser.add_argument('-o', '--output', default='output_font.bmp', help='Output file')
    parser.add_argument('--outline', action='store_true', help='Use outline')
    parser.add_argument('--grid', action='store_true', help='Draw grid')
    
    args = parser.parse_args()
    
    converter = FontConverterCore()
    converter.fontpath = args.font
    converter.fontsize = args.size
    converter.char_width = args.width
    converter.char_height = args.height
    converter.chars_per_row = args.row
    converter.output_path = args.output
    converter.use_outline = args.outline
    converter.draw_grid = args.grid
    
    print("Starting conversion...")
    
    def progress(p):
        print(f"Progress: {p}%", end='\r')
    
    try:
        result = converter.convert(progress)
        print(f"\n\n✓ Conversion completed!")
        print(f"Characters: {result['total_chars']}")
        print(f"Rows: {result['rows']}")
        print(f"Size: {result['width']}x{result['height']}")
        print(f"File: {result['output']}")
    except Exception as e:
        print(f"\n✗ Error: {e}")
        sys.exit(1)


if __name__ == '__main__':
    if len(sys.argv) > 1:
        # CLI mode
        cli_mode()
    else:
        # GUI mode
        app = QApplication(sys.argv)
        window = FontConverterGUI()
        window.show()
        sys.exit(app.exec())
