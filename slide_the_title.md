
I had to change the slides to have same properties throughout.
Slides werent using slide master, so had to go through each slide and move it myself!

When all doors close VBA's door opens !

```
Sub UpdateFirstShapeOnEachSlide()
    Dim slide As slide
    Dim shape As shape
    
    ' Loop through each slide in the presentation
    For Each slide In ActivePresentation.Slides
        ' Get the first shape on the slide
        If slide.Shapes.Count > 0 Then
        
        
       If slide.Shapes(1).HasTextFrame And slide.Shapes(1).TextFrame.HasText Then

            Set shape = slide.Shapes(1)
            ' Change text color to white
            shape.TextFrame.TextRange.Font.Color.RGB = RGB(255, 255, 255)
            ' Move to the top-left corner
            shape.Left = 0
            shape.Top = 0.3 * 28.3465
            ' Align text to the left
            shape.TextFrame.TextRange.ParagraphFormat.Alignment = ppAlignLeft
            shape.TextFrame.TextRange.Font.Size = 30
            shape.Height = 1.43 * 28.3465
            shape.Width = 24.7 * 28.3465
        End If
        
        If slide.Shapes.Count > 1 Then
            With slide.Shapes(2)
                ' Align horizontally (center)
                .Left = (ActivePresentation.PageSetup.SlideWidth - .Width) / 2
                ' Align vertically (center)
                .Top = (ActivePresentation.PageSetup.SlideHeight - .Height) / 2
            End With
        End If
 End If
    Next slide
    
End Sub


```
