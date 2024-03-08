# QGIS-Plugin-I-developed

# MIDTERM     -Abdulsamet Toptaş (21905024)

First of all, the save_attribute file prepared by our course instructor was downloaded.

What is required for Midterm is to draw a line between the furthest and closest points among the coordinates (x and y) points that I called from the dots.shp file that I prepared beforehand. I achieved what I wanted for midterm by adding the code I prepared on top of the existing Python code in this developed plugin.

First of all, I will share the codes assigned to this task in addition to the save_attribute.py file;

 "

 
    from qgis.PyQt.QtWidgets import QFileDialog, QDialog, QMessageBox
    
    from PyQt5.QtCore import QVariant
    
    from qgis.core import QgsPointXY, QgsFeature, QgsGeometry, QgsField, QgsLineString
    
    from qgis.core import QgsWkbTypes
    
    from qgis.core import QgsSimpleLineSymbolLayer, QgsSymbol
    
    from PyQt5.QtGui import QColor
  
      def run(self):  
  
              # Check the geometry type of the selected layer from the user
              if type_of_layer == "point":
                  # Get the selected ID field from the user
                  selected_id_field = self.dlg.comboBox_id.currentText()
      
                  # Get the fields in the QgsVectorLayer object
                  fields = layer.fields()
      
                  # Get the index of the selected field
                  id_field_index = fields.indexFromName(selected_id_field)
      
                  # Calculate the minimum and maximum distances between points and find these points
                  min_distance, max_distance, min_distance_points, max_distance_points = self.calculate_distances(layer, id_field_index)
      
                  # Print the minimum and maximum distances between points
                  print("Minimum Distance:", min_distance)
                  print("Maximum Distance:", max_distance)
      
                  # Draw lines between points at the minimum and maximum distances
                  self.create_line_layer(min_distance_points, "min_distance_line", QgsWkbTypes.LineString, QColor("red"), 1)
                  self.create_line_layer(max_distance_points, "max_distance_line", QgsWkbTypes.LineString, QColor("blue"), 1)
                  
                  layer.startEditing()
                  all_features = layer.getFeatures()
                  
                  for feat in all_features:
                      geom = feat.geometry()
                      x = geom.asPoint().x()
                      y = geom.asPoint().y()
                      
                      # Update X coordinate
                      new_x = {feat.fieldNameIndex('x'): x}
                      layer.dataProvider().changeAttributeValues({feat.id(): new_x })
                      
                      # Update Y coordinate
                      new_y = {feat.fieldNameIndex('y'): y}
                      layer.dataProvider().changeAttributeValues({feat.id(): new_y })
                      
              # Commit changes
              layer.commitChanges()
      
      
      def calculate_distances(self, layer, id_field_index):
          min_distance = float('inf')
          max_distance = float('-inf')
          min_distance_points = None
          max_distance_points = None
      
          features = layer.getFeatures()
      
          for feature1 in features:
              for feature2 in features:
                  if feature1 == feature2:
                      continue
      
                  # Get values in the relevant ID field
                  id1 = feature1.attribute(id_field_index)
                  id2 = feature2.attribute(id_field_index)
      
                  # Get the points
                  point1 = feature1.geometry().asPoint()
                  point2 = feature2.geometry().asPoint()
      
                  # Calculate the distance
                  distance = point1.distance(point2)
      
                  # Check for minimum and maximum distances
                  if distance < min_distance:
                      min_distance = distance
                      min_distance_points = [point1, point2]
      
                  if distance > max_distance:
                      max_distance = distance
                      max_distance_points = [point1, point2]
      
          return min_distance, max_distance, min_distance_points, max_distance_points
      
      
      def create_line_layer(self, points, layer_name, geometry_type, color, thickness):
          # Create a new layer
          line_layer = QgsVectorLayer("LineString?crs=EPSG:4326", layer_name, "memory")
          line_layer.dataProvider().addAttributes([QgsField('id', QVariant.Int)])
          
          # Switch to edit mode
          line_layer.startEditing()
          
          # Create line geometry
          line_geometry = QgsLineString(points)
      
          # Calculate the length of the line in meters
          length_meters = line_geometry.length()
      
          # Write the length to a file or add it to the attribute table
          output_file = "C:/Users/SAMET/Desktop/NOKTALAR/line_lengths.txt"
      
          with open(output_file, 'a') as file:
              file.write(f"{layer_name}: {length_meters:.2f} meters\n")
          
          # Create a new feature
          feature = QgsFeature()
          feature.setGeometry(QgsGeometry(line_geometry))
          feature.setAttributes([1])  # ID
          
          # Add the feature to the layer
          line_layer.dataProvider().addFeatures([feature])
          
          # Exit edit mode
          line_layer.commitChanges()
      
          # Create a line symbol
          line_symbol = QgsSimpleLineSymbolLayer()
          line_symbol.setColor(color)
          line_symbol.setWidth(thickness)
      
          # Create a symbol and add the line symbol
          symbol = QgsSymbol.defaultSymbol(line_layer.geometryType())
          symbol.changeSymbolLayer(0, line_symbol)
      
          # Set the symbol of the layer
          line_layer.renderer().setSymbol(symbol)
      
          # Add the layer to QGIS
          QgsProject.instance().addMapLayer(line_layer)
 "

1. Coordinate Calculations and Creation of Lines:

  - The geometry type of the selected layer is checked and if it is "point":
     - An ID field is selected from the user.
     - By taking the geometry of all features, the minimum and maximum distance between these points is calculated.
     - Lines consisting of points of minimum and maximum distances are created.
     - The X and Y coordinates of each point are added to the layer's properties.
 
2. Auxiliary Function for Distance Calculations:

     - An auxiliary function is used to calculate the minimum and maximum distances between points in a layer.
     - Distances between all points are checked with nested loops.
  
3. Line Layer Creation Function:

     - function create_line_layer creates a specific line layer.
     - Line geometry and length are calculated.
     - The line length is written to a text file.
     - Line is added to the line layer and added to the QGIS project.

**Important Variables:**

- **layer:** Represents the selected vector layer.

- **selected_id_field:** Contains the ID field selected by the user.

- **fields:** Contains all fields in the layer.

- **id_field_index:** Contains the index of the selected ID field.

- **min_distance, max_distance:** Contains the minimum and maximum distance between points.

- **min_distance_points, max_distance_points:** Contains points of minimum and maximum distance.

- **line_layer:** Represents the line layer.

- **line_geometry:** Contains the line geometry.

- **length_meters:** Contains the line length.

- **line_symbol:** Represents the line symbol.

- **symbol:** Represents the symbol of the layer.

The purpose of this code is to calculate the distances between points and the lines belonging to these distances and visualize this information on line layers and text file.



If I were to tell you specifically about the **"def run(self):"** command that I added as an extra;

Assigns the ID field chosen by the user to the selected_id_field variable. It takes the index of the ID field from the fields object and assigns it to the id_field_index variable. Calculates the minimum and maximum distances between points using the calculate_distances function. This function performs the operation using the index of the layer and ID field. Prints the calculated minimum and maximum distances to the console. Creates line layers between points with minimum and maximum distances using the create_line_layer function. Lines are visualized with the specified color and thickness (QColor("red"), 1 and QColor("blue"), 1).

If I were to explain especially the **"def calculate_distances(self, layer, id_field_index):"** command that I added extra;

The calculate_distances function takes a vector layer and the index of an ID field (id_field_index) as parameters. The initial values of the minimum and maximum distances are assigned to infinity (float('inf') and float('-inf')). These values will be used in comparisons in the loops. All features (points) in the layer are accessed through the features variable. The distance between both points is calculated using two nested loops. Two different points are selected (feature1 and feature2), and the ID values of these points are taken and compared. The distance between identical features is not calculated (with the check feature1 == feature2). The distance between two points is calculated with point1.distance(point2). The calculated distance is compared with the minimum and maximum distance values. If a new minimum or maximum is found, these values are updated and the corresponding points (min_distance_points and max_distance_points) are stored. Once the two loops are completed, the function returns a tuple containing the minimum and maximum distances, and the points for those distances.

Briefly, this function calculates the minimum and maximum distance between points in the given vector layer. These distances and the points belonging to these distances are returned as a tuple.


If I were to explain especially the **"def create_line_layer(self, points, layer_name, geometry_type, color, thickness):"** command that I added extra;

The create_line_layer function initially creates a "memory" layer and defines the geometry and properties of this layer. First, a new line layer (line_layer) is created in memory using the QgsVectorLayer class and the geometry type of this layer is determined as "LineString". Then, a property called "id" is added to the layer's data provider to store the line length.

Next, the function creates a line geometry from the given list of points and calculates the line length using this geometry. The calculated length is stored in the length_meters variable as a value in meters. This length information is added to a specified file ("C:/Users/SAMET/Desktop/POTS/line_lengths.txt") so that the line lengths are recorded.

Next, a feature associated with the line geometry is created and this feature is added to the line layer. The line length and an ID value (set as just "1" in this example) are assigned to this property. Line geometry and this property is used to define the symbol and properties of the line layer.

Finally, the line symbol is created and this symbol determines the appearance of the line layer. The created symbol replaces the symbol of the line layer and visualizes the line with the specified color and thickness values. This way, the line layer is added to the QGIS project and the line lengths are stored in the user-specified file.
