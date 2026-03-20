# ExcalidrawAutomate Full API Reference

Based on the official TypeScript definitions from [obsidian-excalidraw-plugin v2.21.x](https://github.com/zsviczian/obsidian-excalidraw-plugin).

Read this file when you need the exact signature for a less common method. The SKILL.md covers the most-used methods; this file is the comprehensive reference.

## Table of Contents

1. [Initialization & Lifecycle](#initialization--lifecycle)
2. [Element Creation](#element-creation)
3. [Element Manipulation](#element-manipulation)
4. [Grouping & Frames](#grouping--frames)
5. [Style Configuration](#style-configuration)
6. [View Operations](#view-operations)
7. [File & Scene Operations](#file--scene-operations)
8. [Export & Rendering](#export--rendering)
9. [Measurement & Analysis](#measurement--analysis)
10. [Color Utilities](#color-utilities)
11. [Image Operations](#image-operations)
12. [Event Hooks](#event-hooks)
13. [Script Settings](#script-settings)
14. [Miscellaneous Utilities](#miscellaneous-utilities)

---

## Initialization & Lifecycle

```typescript
// Access the singleton
const ea = window.ExcalidrawAutomate;

// Reset to clean defaults — call before every script
ea.reset(): void

// Clear all elements from workbench (but keep styles)
ea.clear(): void

// Destroy the EA instance (rarely needed)
ea.destroy(): void

// Check plugin version compatibility
ea.verifyMinimumPluginVersion(version: string): boolean
```

---

## Element Creation

All creation methods return a `string` element ID.

### Shapes

```typescript
addRect(topX: number, topY: number, width: number, height: number, id?: string): string
addEllipse(topX: number, topY: number, width: number, height: number, id?: string): string
addDiamond(topX: number, topY: number, width: number, height: number, id?: string): string
addBlob(topX: number, topY: number, width: number, height: number, id?: string): string
addFrame(topX: number, topY: number, width: number, height: number, name?: string): string
```

### Text

```typescript
addText(
  topX: number,
  topY: number,
  text: string,
  formatting?: {
    width?: number;
    height?: number;
    textAlign?: "left" | "center" | "right";
    verticalAlign?: "top" | "middle";
    box?: boolean | "box" | "blob" | "ellipse" | "diamond";
    boxPadding?: number;
    boxStrokeColor?: string;
  },
  id?: string
): string
```

When `box` is `true` or a shape name, the text is wrapped in that shape. The shape inherits current `ea.style` settings.

### Lines and Arrows

```typescript
addLine(points: [number, number][], id?: string): string

addArrow(
  points: [number, number][],
  formatting?: {
    startArrowHead?: "arrow" | "bar" | "dot" | "none";
    endArrowHead?: "arrow" | "bar" | "dot" | "none";
  },
  id?: string
): string
```

Points are relative coordinates: `[[0,0], [100,50], [200,0]]` draws a line from origin through two more points.

### Connections

```typescript
connectObjects(
  objectA: string,       // element ID
  connectionA: "top" | "right" | "bottom" | "left",
  objectB: string,       // element ID
  connectionB: "top" | "right" | "bottom" | "left",
  formatting?: {
    numberOfPoints?: number;
    startArrowHead?: "arrow" | "bar" | "dot" | "none";
    endArrowHead?: "arrow" | "bar" | "dot" | "none";
    padding?: number;
  }
): void

// Add a text label to an existing line/arrow
addLabelToLine(lineId: string, label: string): void

// Connect an element to whatever is selected in the view
connectObjectWithViewSelectedElement(
  objectA: string,
  connectionA: "top" | "right" | "bottom" | "left",
  connectionB: "top" | "right" | "bottom" | "left",
  formatting?: object
): void
```

### Special Elements

```typescript
addImage(
  topX: number | AddImageOptions,
  topY: number,
  imageFile: TFile | string,
  scale?: boolean,
  anchor?: boolean
): Promise<string>

addLaTex(
  topX: number,
  topY: number,
  tex: string,
  scaleX?: number,
  scaleY?: number
): Promise<string>

addMermaid(
  diagram: string,
  groupElements?: boolean
): Promise<string[] | string>

addIFrame(
  topX: number,
  topY: number,
  width: number,
  height: number,
  url?: string,
  file?: TFile
): string

addEmbeddable(
  topX: number,
  topY: number,
  width: number,
  height: number,
  url?: string,
  file?: TFile,
  embeddableCustomData?: EmbeddableMDCustomProps
): string
```

---

## Element Manipulation

```typescript
// Get element from workbench by ID
getElement(id: string): Mutable<ExcalidrawElement>

// Get all workbench elements
getElements(): Mutable<ExcalidrawElement>[]

// Clone an element (returns a new element with new ID)
cloneElement(element: ExcalidrawElement): ExcalidrawElement

// Generate a new unique element ID
generateElementId(): string

// Refresh text dimensions after changing content
refreshTextElementSize(id: string): void

// Change z-index of an element in the view
moveViewElementToZIndex(elementId: string, index: number): void

// Add/update custom data on an element
addAppendUpdateCustomData(id: string, newData: Partial<Record<string, unknown>>): ExcalidrawElement
```

---

## Grouping & Frames

```typescript
// Group elements — returns group ID
addToGroup(objectIds: string[]): string

// Add elements into a frame
addElementsToFrame(frameId: string, elementIDs: string[]): void

// Find the common group for a set of elements
getCommonGroupForElements(elementIds: string[]): string | null

// Get all elements sharing a group with the given element
getElementsInTheSameGroupWithElement(elementId: string): ExcalidrawElement[]

// Get all elements inside a frame
getElementsInFrame(frameId: string): ExcalidrawElement[]

// Get the maximum number of groups
getMaximumGroups(): number
```

---

## Style Configuration

Set on `ea.style` before creating elements:

```typescript
ea.style.strokeColor: string          // "#1e1e1e"
ea.style.backgroundColor: string      // "transparent", "#hex", or color name
ea.style.fillStyle: string            // "hachure" | "cross-hatch" | "solid"
ea.style.strokeWidth: number          // line thickness
ea.style.strokeStyle: string          // "solid" | "dashed" | "dotted"
ea.style.roughness: number            // 0=Architect, 1=Artist, 2=Cartoonist
ea.style.opacity: number              // 0-100
ea.style.fontSize: number             // pixels
ea.style.fontFamily: number           // 1=Hand-drawn, 2=Normal, 3=Monospace
ea.style.textAlign: string            // "left" | "center" | "right"
ea.style.verticalAlign: string        // "top" | "middle"
ea.style.startArrowHead: string       // "arrow" | "bar" | "dot" | "none"
ea.style.endArrowHead: string         // "arrow" | "bar" | "dot" | "none"
ea.style.angle: number                // radians
```

Setter helpers (use numeric indices):
```typescript
setFillStyle(val: number): "hachure" | "cross-hatch" | "solid"
setStrokeStyle(val: number): "solid" | "dashed" | "dotted"
setStrokeSharpness(val: number): "round" | "sharp"
setFontFamily(val: number): string    // 1, 2, or 3
setTheme(val: number): "light" | "dark"
```

Canvas settings:
```typescript
ea.canvas.theme: "light" | "dark"
ea.canvas.viewBackgroundColor: string
ea.canvas.gridSize: number
ea.canvas.gridMode: boolean
```

---

## View Operations

```typescript
// Set the target view for subsequent operations
setView(view: ExcalidrawView): void

// Get the Excalidraw API from the active view
getExcalidrawAPI(): ExcalidrawAPI

// Get all elements in the active view
getViewElements(): ExcalidrawElement[]

// Get selected element(s)
getViewSelectedElement(): ExcalidrawElement
getViewSelectedElements(): ExcalidrawElement[]

// Get the last pointer position in the view
getViewLastPointerPosition(): { x: number; y: number }

// Add workbench elements to the view
addElementsToView(
  elements?: ExcalidrawElement[],
  addWatermark?: boolean,
  openInReadMode?: boolean
): Promise<void>

// Update the scene
viewUpdateScene(scene: object): Promise<void>

// Delete elements from view
deleteViewElements(elementIds: string[]): void

// Select elements in view
selectElementsInView(elementIds: string[]): void

// Zoom to fit specific elements
viewZoomToElements(elementIds: string[]): void

// Toggle fullscreen
viewToggleFullScreen(): void

// Toggle view mode (read-only)
setViewModeEnabled(enabled: boolean): void

// Copy view elements to EA workbench for editing
copyViewElementsToEAforEditing(): void

// Register/deregister this EA instance as the view's EA
registerThisAsViewEA(): void
deregisterThisAsViewEA(): void
```

---

## File & Scene Operations

```typescript
// Create a new drawing file in the vault
create(params?: {
  filename?: string;
  foldername?: string;
  templatePath?: string;
  onNewPane?: boolean;
}): Promise<string>   // returns file path

// Load a scene from an existing file
getSceneFromFile(file: TFile): Promise<{
  elements: ExcalidrawElement[];
  appState: AppState;
}>

// Check if a file is an Excalidraw file
isExcalidrawFile(file?: TFile): boolean

// Check if a view is an Excalidraw view
isExcalidrawView(view?: any): boolean

// Check if file is an Excalidraw mask file
isExcalidrawMaskFile(file?: TFile): boolean

// Open a file in a new or adjacent leaf
openFileInNewOrAdjacentLeaf(file: TFile, targetPane?: PaneTarget): void

// Get a new unique filepath (avoids collisions)
getNewUniqueFilepath(filename: string, folderpath: string): string

// Check and create folder if it doesn't exist
checkAndCreateFolder(folderpath: string): Promise<TFolder>

// Get attachment filepath
getAttachmentFilepath(filename: string): Promise<string>

// Get list of template files
getListOfTemplateFiles(): TFile[] | null

// Get embedded images file tree
getEmbeddedImagesFiletree(excalidrawFile?: TFile): TFile[]
```

---

## Export & Rendering

```typescript
// Export to SVG
createSVG(
  templatePath?: string,
  embedFont?: boolean,
  exportSettings?: ExportSettings,
  loader?: EmbeddedFilesLoader,
  theme?: string,
  padding?: number
): Promise<SVGSVGElement>

// Export to PNG (returns blob or data)
createPNG(
  templatePath?: string,
  scale?: number,
  exportSettings?: ExportSettings,
  loader?: EmbeddedFilesLoader,
  theme?: string,
  padding?: number
): Promise<any>

// Export to PNG as base64 string
createPNGBase64(...same params as createPNG): Promise<string>

// Export to PDF
createPDF(): Promise<void>

// Copy drawing to clipboard
toClipboard(templatePath?: string): Promise<void>

// Import SVG into Excalidraw elements
importSVG(svgString: string): Promise<string[]>

// Convert TeX to data URL
tex2dataURL(
  tex: string,
  scale?: number
): Promise<{
  mimeType: MimeType;
  fileId: FileId;
  dataURL: DataURL;
  created: number;
  size: { width: number; height: number };
}>
```

---

## Measurement & Analysis

```typescript
// Measure text dimensions
measureText(text: string, options?: {
  fontSize?: number;
  fontFamily?: number;
}): { width: number; height: number }

// Get bounding box of elements
getBoundingBox(elementIds?: string[]): {
  x: number;
  y: number;
  width: number;
  height: number;
}

// Get the largest element
getLargestElement(): ExcalidrawElement

// Test intersection of element with a line
intersectElementWithLine(
  element: ExcalidrawElement,
  line: [Point, Point]
): Point[] | null

// Wrap text to a character width
wrapText(text: string, lineLen: number): string
```

---

## Color Utilities

```typescript
hexStringToRgb(hex: string): [number, number, number]
rgbToHexString(r: number, g: number, b: number): string
hslToRgb(h: number, s: number, l: number): [number, number, number]
rgbToHsl(r: number, g: number, b: number): [number, number, number]
colorNameToHex(name: string): string
getCM(): ColorMaster
getPolyBool(): PolyBool
```

---

## Image Operations

```typescript
// Get the vault file backing an image element
getViewFileForImageElement(imageElementId: string): TFile | null

// Get/set color map for SVG image recoloring
getColorMapForImageElement(imageElementId: string): ColorMap
updateViewSVGImageColorMap(imageElementId: string, colorMap: ColorMap): void
getSVGColorInfoForImgElement(imageElementId: string): SVGColorInfo

// Get colors used in an Excalidraw file
getColorsFromExcalidrawFile(file: TFile): string[]
getColorsFromSVGString(svgString: string): string[]

// Get original image dimensions
getOriginalImageSize(imageFile: TFile | string): Promise<{ width: number; height: number }>

// Reset image to original aspect ratio
resetImageAspectRatio(imageElementId: string): void
```

---

## Event Hooks

Register hooks to intercept plugin behavior. Return `true` to prevent default behavior, `false` to allow it.

```typescript
// Intercept file creation
ea.onFileCreateHook = (data: { ea, excalidrawFile, view }) => boolean

// Handle drag-and-drop
ea.onDropHook = (data: { ea, event, draggable, view, pointerPosition }) => boolean

// Custom link click behavior
ea.onLinkClickHook = (element, link, event, view) => boolean

// Handle paste events
ea.onPasteHook = (data: { ea, event, view, pointerPosition }) => boolean

// React to view mode changes
ea.onViewModeChangeHook = (isViewModeEnabled, view) => boolean
```

---

## Script Settings

Persist settings across Obsidian sessions:

```typescript
ea.getScriptSettings(): Record<string, any>
ea.setScriptSettings(settings: Record<string, any>): void
```

---

## Miscellaneous Utilities

```typescript
// Access the Obsidian module
ea.obsidian: ObsidianModule

// Post to OpenAI (for AI integration scripts)
ea.postOpenAI(request: AIRequest): Promise<RequestUrlResponse>

// Extract code blocks from markdown
ea.extractCodeBlocks(markdown: string): Array<{ data: string; type: string }>

// Convert string to data URL
ea.convertStringToDataURL(data: string, type?: string): Promise<string>

// Compress/decompress for storage
ea.compressToBase64(str: string): string
ea.decompressFromBase64(data: string): string

// Get active embeddable view or editor
ea.getActiveEmbeddableViewOrEditor(view?: ExcalidrawView):
  { view: any } | { file: TFile; editor: Editor } | null

// Add back-of-card note to view
ea.addBackOfTheCardNoteToView(...): void

// Get embedded files loader
ea.getEmbeddedFilesLoader(): EmbeddedFilesLoader

// Get export settings
ea.getExportSettings(): ExportSettings

// Print startup timing breakdown
ea.printStartupBreakdown(): void

// Get help for a function
ea.help(target: Function | string): void
```

---

## Sources

- [Official API Introduction](https://zsviczian.github.io/obsidian-excalidraw-plugin/API/introduction.html)
- [TypeScript Definitions](https://github.com/zsviczian/obsidian-excalidraw-plugin/blob/master/docs/API/ExcalidrawAutomate.d.ts)
- [AutomateHowTo.md](https://github.com/zsviczian/obsidian-excalidraw-plugin/blob/master/AutomateHowTo.md)
- [DeepWiki: Automation and Scripting](https://deepwiki.com/zsviczian/obsidian-excalidraw-plugin/4-automation-and-scripting)
- [Script Engine Documentation](https://github.com/zsviczian/obsidian-excalidraw-plugin/blob/master/docs/ExcalidrawScriptsEngine.md)
