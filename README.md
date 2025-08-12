# Image-Processing2
Finding Angles


import cv2
import math
import numpy as np
import pandas as pd
from openpyxl import load_workbook

# List of image paths to be loaded
image_paths = [
    "nlt1.jpg",
    "nlt2.jpg",
    "nrt1.jpg",
    "nrt2.jpg",
    "nrt3.jpg",
    "nrt4.jpg",
    "nrt5.jpg",
    "nrt6.jpg",
    "nrt7.jpg" 
]


# Load and resize all images
scale_percent = 30
white_box_width = 275
white_box_height = 200 # New variable for the height of the white box


# Function to load and resize images
def load_and_resize_images(image_paths, scale_percent):
    images = []
    for path in image_paths:
        img_original = cv2.imread(path)
        width = int(img_original.shape[1] * scale_percent / 100)
        height = int(img_original.shape[0] * scale_percent / 100)
        dim = (width, height)
        img_resized = cv2.resize(img_original, dim, interpolation=cv2.INTER_AREA)
        images.append(img_resized)
    return images

images = load_and_resize_images(image_paths, scale_percent)

# Combine multiple images into a single canvas
def combine_images(images, white_box_width, white_box_height):
    # Find maximum height and total width for the combined image
    max_height = max(img.shape[0] for img in images) + white_box_height
    total_width = sum(img.shape[1] for img in images) + white_box_width

    # Create a white canvas
    canvas = np.full((max_height, total_width, 3), 255, dtype=np.uint8)
    x_offset = 0

    # Place each image on the canvas
    for img in images:
        canvas[:img.shape[0], x_offset:x_offset + img.shape[1]] = img
        x_offset += img.shape[1]

    return canvas

combined_canvas = combine_images(images, white_box_width, white_box_height)

# Initialize lists to store points and angles
points_list = []
inside_angles = []
outside_angles = []
circle_number = 1

# Mouse callback function to handle clicks
def mouse_points(event, x, y, flags, params):
    global circle_number

    if event == cv2.EVENT_LBUTTONDOWN:
        # Adjust the click coordinates to match the original image size
        adjusted_x = int(x / zoom_factor)
        adjusted_y = int(y / zoom_factor)

        size = len(points_list)
        if size % 3 != 0:
            # Draw line at the adjusted coordinates
            cv2.line(combined_canvas, tuple(points_list[round((size-1)/3)*3]), (adjusted_x, adjusted_y), (255, 0, 0), 2)
        else:
            # Draw circle at the adjusted coordinates
            cv2.circle(combined_canvas, (adjusted_x, adjusted_y), 10, (0, 0, 255), cv2.FILLED)
            cv2.putText(combined_canvas, str(circle_number), (adjusted_x - 5, adjusted_y + 5), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 2)
            circle_number += 1

        # Store the adjusted point in the points list
        points_list.append([adjusted_x, adjusted_y])

        if len(points_list) % 3 == 0:
            calculate_angle(points_list)

# Function to calculate angles from three points
def calculate_angle(points_list):
    pt1, pt2, pt3 = points_list[-3:]
    angle = get_angle(pt1, pt2, pt3)
    inside_angle = min(angle, 360 - angle)
    outside_angle = 360 - inside_angle
    inside_angles.append(inside_angle)
    outside_angles.append(outside_angle)
    average_inside_angle = sum(inside_angles) / len(inside_angles)
    average_outside_angle = sum(outside_angles) / len(outside_angles)
    update_white_box(inside_angles, average_inside_angle)
    save_to_excel(inside_angles, outside_angles, average_inside_angle, average_outside_angle)

# Function to calculate the angle between two lines formed by three points
def get_angle(pt1, pt2, pt3):
    vector1 = (pt2[0] - pt1[0], pt2[1] - pt1[1])
    vector2 = (pt3[0] - pt1[0], pt3[1] - pt1[1])
    dot_product = vector1[0] * vector2[0] + vector1[1] * vector2[1]
    magnitude1 = math.sqrt(vector1[0] ** 2 + vector1[1] ** 2)
    magnitude2 = math.sqrt(vector2[0] ** 2 + vector2[1] ** 2)
    if magnitude1 == 0 or magnitude2 == 0:
        return 0
    cos_angle = dot_product / (magnitude1 * magnitude2)
    angle = math.degrees(math.acos(max(-1, min(1, cos_angle))))
    return angle

# Function to update the white box with angles
def update_white_box(angles, average_inside_angle):
    # Clear the white box area
    combined_canvas[:, combined_canvas.shape[1] - white_box_width:] = 255
    
    # Write each angle on the white box
    for i, angle in enumerate(angles):
        text = f"Angle {i+1}: {angle:.2f} deg"  # Changed '°' to 'deg'
        cv2.putText(combined_canvas, text, (combined_canvas.shape[1] - white_box_width + 10, 30 + i * 30), cv2.FONT_HERSHEY_SIMPLEX, 0.45, (0, 0, 0), 2)
    
    # Write the average of angles on the white box
    cv2.putText(combined_canvas, f"AoA: {average_inside_angle:.2f} deg",  # Changed '°' to 'deg'
                (combined_canvas.shape[1] - white_box_width + 10, 30 + len(angles) * 30 + 30), 
                cv2.FONT_HERSHEY_SIMPLEX, 0.55, (0, 0, 255), 2)

# Function to save angles to Excel and merge columns
def save_to_excel(inside_angles, outside_angles, average_inside_angle, average_outside_angle):
    data = {
        'No': list(range(1, len(inside_angles) + 1)),
        'Inside Angles (°)': inside_angles,
        'Outside Angles (°)': outside_angles,
        'Average Inside Angle (°)': [average_inside_angle] * len(inside_angles),
        'Average Outside Angle (°)': [average_outside_angle] * len(outside_angles)
    }
    df = pd.DataFrame(data)
    excel_file_path = "angles_output.xlsx"
    df.to_excel(excel_file_path, index=False)
    workbook = load_workbook(excel_file_path)
    sheet = workbook.active
    sheet.merge_cells(start_row=2, start_column=4, end_row=len(inside_angles) + 1, end_column=4)
    sheet.merge_cells(start_row=2, start_column=5, end_row=len(outside_angles) + 1, end_column=5)
    workbook.save(excel_file_path)
    print("Data saved to 'angles_output.xlsx' with merged columns for averages")

# Function to handle zoom in/out
def zoom_image(img, zoom_factor):
    width = int(img.shape[1] * zoom_factor)
    height = int(img.shape[0] * zoom_factor)
    dim = (width, height)
    return cv2.resize(img, dim, interpolation=cv2.INTER_LINEAR)


# Main loop
zoom_factor = 1.0  # Initial zoom factor
zoom_step = 0.1  # Step for zooming in/out

while True:
    zoomed_img = zoom_image(combined_canvas, zoom_factor)
    cv2.imshow("Combined Images", zoomed_img)
    cv2.setMouseCallback("Combined Images", mouse_points)
    cv2.imwrite('C:/Users/Asus/Exercise/Via_Angles.png', zoomed_img)

    key = cv2.waitKey(1) & 0xFF
    
    if key == ord('+') or key == ord('='):
        zoom_factor += zoom_step
    elif key == ord('-'):
        zoom_factor = max(zoom_factor - zoom_step, 0.1)
    elif key == ord('q'):
        print("Exiting...")
        break

cv2.destroyAllWindows()
