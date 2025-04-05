# Project Code Overview
## kotlin File: `RecorderManagerTest.kt`
```kotlin
package com.example.gamemapper

import android.content.Context
import android.net.Uri
import android.view.KeyEvent
import androidx.test.core.app.ApplicationProvider
import org.junit.Assert.assertEquals
import org.junit.Assert.assertTrue
import org.junit.Before
import org.junit.Test
import java.io.File

class RecorderManagerTest {

    private lateinit var context: Context

    @Before
    fun setup() {
        context = ApplicationProvider.getApplicationContext()
    }

    @Test
    fun testRecording() {
        RecorderManager.startRecording()
        RecorderManager.recordAction(KeyEvent.KEYCODE_A, 100f, 200f)
        RecorderManager.stopRecording()
        assertEquals(1, RecorderManager.getRecordedActions().size)
        val action = RecorderManager.getRecordedActions()[0]
        assertEquals(KeyEvent.KEYCODE_A, action.keyCode)
        assertEquals(100f, action.x)
        assertEquals(200f, action.y)
    }

    @Test
    fun testSaveAndLoad() {
        RecorderManager.startRecording()
        RecorderManager.recordAction(KeyEvent.KEYCODE_B, 150f, 250f)
        RecorderManager.stopRecording()

        val file = File(context.cacheDir, "test.txt")
        val uri = Uri.fromFile(file)
        assertTrue(RecorderManager.saveRecording(context, uri))

        RecorderManager.clearPool()
        assertTrue(RecorderManager.loadRecording(context, uri))
        assertEquals(1, RecorderManager.getRecordedActions().size)
    }
}
```

## kotlin File: `MappingServiceTest.kt`
```kotlin
package com.example.gamemapper

import android.content.Context
import android.view.KeyEvent
import androidx.test.core.app.ApplicationProvider
import org.junit.Assert.assertEquals
import org.junit.Before
import org.junit.Test

class MappingServiceTest {

    private lateinit var service: MappingService

    @Before
    fun setup() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        service = MappingService()
        service.onServiceConnected()
    }

    @Test
    fun testKeyMappings() {
        assertEquals(7, service.keyMappings.size)
        assertEquals(Pair(500f, 300f), service.keyMappings[KeyEvent.KEYCODE_W])
    }
}
```

## kotlin File: `ButtonSettingsDialog.kt`
```kotlin
package com.example.gamemapper

import android.app.AlertDialog
import android.content.Context
import android.graphics.Color
import android.view.LayoutInflater
import android.widget.Button
import android.widget.RadioButton
import android.widget.RadioGroup
import android.widget.SeekBar
import android.widget.TextView
import android.widget.Toast
import android.view.View

class ButtonSettingsDialog(
    private val context: Context,
    private val button: CustomOverlayButton,
    private val onSettingsChanged: (CustomOverlayButton) -> Unit
) {
    private lateinit var dialog: AlertDialog
    private lateinit var colorPreview: View
    private lateinit var sizeValueText: TextView
    private lateinit var transparencyValueText: TextView
    
    fun show() {
        val dialogView = LayoutInflater.from(context).inflate(R.layout.dialog_button_settings, null)
        
        // Инициализация элементов управления
        val shapeRadioGroup = dialogView.findViewById<RadioGroup>(R.id.shapeRadioGroup)
        val shapeCircle = dialogView.findViewById<RadioButton>(R.id.shapeCircle)
        val shapeSquare = dialogView.findViewById<RadioButton>(R.id.shapeSquare)
        val shapeRounded = dialogView.findViewById<RadioButton>(R.id.shapeRounded)
        colorPreview = dialogView.findViewById(R.id.colorPreview)
        val selectColorButton = dialogView.findViewById<Button>(R.id.selectColorButton)
        val sizeSeekBar = dialogView.findViewById<SeekBar>(R.id.sizeSeekBar)
        sizeValueText = dialogView.findViewById(R.id.sizeValueText)
        val transparencySeekBar = dialogView.findViewById<SeekBar>(R.id.transparencySeekBar)
        transparencyValueText = dialogView.findViewById(R.id.transparencyValueText)
        
        // Установка текущих значений
        when (button.buttonShape) {
            CustomOverlayButton.SHAPE_CIRCLE -> shapeCircle.isChecked = true
            CustomOverlayButton.SHAPE_SQUARE -> shapeSquare.isChecked = true
            CustomOverlayButton.SHAPE_ROUNDED -> shapeRounded.isChecked = true
        }
        
        colorPreview.setBackgroundColor(button.buttonColor)
        sizeSeekBar.progress = button.buttonSize.toInt()
        sizeValueText.text = button.buttonSize.toInt().toString()
        transparencySeekBar.progress = button.buttonAlpha
        updateTransparencyText(button.buttonAlpha)
        
        // Обработчики событий
        shapeRadioGroup.setOnCheckedChangeListener { _, checkedId ->
            when (checkedId) {
                R.id.shapeCircle -> button.buttonShape = CustomOverlayButton.SHAPE_CIRCLE
                R.id.shapeSquare -> button.buttonShape = CustomOverlayButton.SHAPE_SQUARE
                R.id.shapeRounded -> button.buttonShape = CustomOverlayButton.SHAPE_ROUNDED
            }
            button.invalidate()
            onSettingsChanged(button)
        }
        
        selectColorButton.setOnClickListener {
            showColorPickerDialog()
        }
        
        sizeSeekBar.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
            override fun onProgressChanged(seekBar: SeekBar, progress: Int, fromUser: Boolean) {
                val size = maxOf(progress, 60) // Минимальный размер 60
                sizeValueText.text = size.toString()
                button.buttonSize = size.toFloat()
                button.requestLayout()
                onSettingsChanged(button)
            }
            
            override fun onStartTrackingTouch(seekBar: SeekBar) {}
            
            override fun onStopTrackingTouch(seekBar: SeekBar) {}
        })
        
        transparencySeekBar.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
            override fun onProgressChanged(seekBar: SeekBar, progress: Int, fromUser: Boolean) {
                button.buttonAlpha = progress
                updateTransparencyText(progress)
                button.invalidate()
                onSettingsChanged(button)
            }
            
            override fun onStartTrackingTouch(seekBar: SeekBar) {}
            
            override fun onStopTrackingTouch(seekBar: SeekBar) {}
        })
        
        dialog = AlertDialog.Builder(context)
            .setTitle(R.string.button_settings)
            .setView(dialogView)
            .setPositiveButton(R.string.ok) { _, _ ->
                // Настройки уже применены
            }
            .create()
            
        dialog.show()
    }
    
    private fun updateTransparencyText(alpha: Int) {
        val percentage = (alpha * 100 / 255)
        transparencyValueText.text = "$percentage%"
    }
    
    private fun showColorPickerDialog() {
        val colors = arrayOf(
            Color.parseColor("#80FFFFFF"), // Белый полупрозрачный
            Color.parseColor("#80FF0000"), // Красный полупрозрачный
            Color.parseColor("#8000FF00"), // Зеленый полупрозрачный
            Color.parseColor("#800000FF"), // Синий полупрозрачный
            Color.parseColor("#80FFFF00"), // Желтый полупрозрачный
            Color.parseColor("#80FF00FF"), // Пурпурный полупрозрачный
            Color.parseColor("#8000FFFF")  // Голубой полупрозрачный
        )
        
        val colorViews = Array(colors.size) { i ->
            View(context).apply {
                setBackgroundColor(colors[i])
                layoutParams = android.view.ViewGroup.LayoutParams(50, 50)
                setPadding(5, 5, 5, 5)
            }
        }
        
        val colorNames = arrayOf(
            "Белый", "Красный", "Зеленый", "Синий", "Желтый", "Пурпурный", "Голубой"
        )
        
        AlertDialog.Builder(context)
            .setTitle(R.string.button_color)
            .setItems(colorNames) { _, which ->
                button.buttonColor = colors[which]
                colorPreview.setBackgroundColor(colors[which])
                button.invalidate()
                onSettingsChanged(button)
            }
            .show()
    }
}

```

## kotlin File: `CustomOverlayButton.kt`
```kotlin
package com.example.gamemapper

import android.content.Context
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Paint
import android.graphics.RectF
import android.view.View

class CustomOverlayButton(context: Context) : View(context) {
    var buttonSize: Float = 120f
    var buttonColor: Int = Color.parseColor("#80FFFFFF")
    var buttonShape: Int = SHAPE_CIRCLE
    var buttonText: String = ""
    var buttonTextColor: Int = Color.BLACK
    var buttonBorderColor: Int = Color.BLACK
    var buttonBorderWidth: Float = 0f
    var buttonAlpha: Int = 128 // 0-255
    
    private val paint = Paint(Paint.ANTI_ALIAS_FLAG)
    private val textPaint = Paint(Paint.ANTI_ALIAS_FLAG)
    private val borderPaint = Paint(Paint.ANTI_ALIAS_FLAG)
    private val rect = RectF()
    
    companion object {
        const val SHAPE_CIRCLE = 0
        const val SHAPE_SQUARE = 1
        const val SHAPE_ROUNDED = 2
        const val SHAPE_TRIANGLE = 3
        const val SHAPE_DIAMOND = 4
    }
    
    init {
        textPaint.color = buttonTextColor
        textPaint.textSize = 24f
        textPaint.textAlign = Paint.Align.CENTER
        
        borderPaint.style = Paint.Style.STROKE
        borderPaint.color = buttonBorderColor
        borderPaint.strokeWidth = buttonBorderWidth
    }
    
    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        
        // Устанавливаем прозрачность
        paint.alpha = buttonAlpha
        paint.color = buttonColor
        
        val centerX = width / 2f
        val centerY = height / 2f
        val radius = Math.min(width, height) / 2f - 2f
        
        when (buttonShape) {
            SHAPE_CIRCLE -> {
                canvas.drawCircle(centerX, centerY, radius, paint)
                if (buttonBorderWidth > 0) {
                    canvas.drawCircle(centerX, centerY, radius, borderPaint)
                }
            }
            SHAPE_SQUARE -> {
                rect.set(2f, 2f, width - 2f, height - 2f)
                canvas.drawRect(rect, paint)
                if (buttonBorderWidth > 0) {
                    canvas.drawRect(rect, borderPaint)
                }
            }
            SHAPE_ROUNDED -> {
                rect.set(2f, 2f, width - 2f, height - 2f)
                canvas.drawRoundRect(rect, 15f, 15f, paint)
                if (buttonBorderWidth > 0) {
                    canvas.drawRoundRect(rect, 15f, 15f, borderPaint)
                }
            }
            SHAPE_TRIANGLE -> {
                val path = android.graphics.Path()
                path.moveTo(centerX, 2f)
                path.lineTo(width - 2f, height - 2f)
                path.lineTo(2f, height - 2f)
                path.close()
                canvas.drawPath(path, paint)
                if (buttonBorderWidth > 0) {
                    canvas.drawPath(path, borderPaint)
                }
            }
            SHAPE_DIAMOND -> {
                val path = android.graphics.Path()
                path.moveTo(centerX, 2f)
                path.lineTo(width - 2f, centerY)
                path.lineTo(centerX, height - 2f)
                path.lineTo(2f, centerY)
                path.close()
                canvas.drawPath(path, paint)
                if (buttonBorderWidth > 0) {
                    canvas.drawPath(path, borderPaint)
                }
            }
        }
        
        // Рисуем текст в центре кнопки
        textPaint.color = buttonTextColor
        val textHeight = textPaint.descent() - textPaint.ascent()
        val textOffset = textHeight / 2 - textPaint.descent()
        canvas.drawText(buttonText, centerX, centerY + textOffset, textPaint)
    }
    
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        val size = buttonSize.toInt()
        setMeasuredDimension(size, size)
    }
}

```

## kotlin File: `EditMappingActivity.kt`
```kotlin
package com.example.gamemapper

import android.content.Context
import android.os.Bundle
import android.view.KeyEvent
import android.widget.Button
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class EditMappingActivity : AppCompatActivity() {
    private lateinit var adapter: KeyMappingAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_edit_mapping)

        val keyList = findViewById<RecyclerView>(R.id.keyList)
        val addButton = findViewById<Button>(R.id.addButton)
        val mappings = keyMappingsFromService().toMutableMap()
        adapter = KeyMappingAdapter(mappings) { keyCode, x, y ->
            mappings[keyCode] = Pair(x, y)
            saveOverlayPositions(mappings)
            updateServiceMappings(mappings)
        }
        keyList.adapter = adapter
        keyList.layoutManager = LinearLayoutManager(this)

        addButton.setOnClickListener {
            mappings[KeyEvent.KEYCODE_UNKNOWN] = Pair(0f, 0f)
            adapter.notifyItemInserted(mappings.size - 1)
            Toast.makeText(this, "New mapping added", Toast.LENGTH_SHORT).show()
        }
    }

    private fun keyMappingsFromService(): Map<Int, Pair<Float, Float>> {
        return (getSystemService(Context.ACCESSIBILITY_SERVICE) as? MappingService)?.keyMappings ?: emptyMap()
    }

    private fun saveOverlayPositions(mappings: Map<Int, Pair<Float, Float>>) {
        val prefs = getSharedPreferences("GameMapperPrefs", Context.MODE_PRIVATE)
        with(prefs.edit()) {
            for ((keyCode, position) in mappings) {
                putFloat("x_$keyCode", position.first)
                putFloat("y_$keyCode", position.second)
            }
            apply()
        }
    }

    private fun updateServiceMappings(mappings: Map<Int, Pair<Float, Float>>) {
        val service = getSystemService(Context.ACCESSIBILITY_SERVICE) as? MappingService
        service?.keyMappings?.putAll(mappings)
    }
}
```

## kotlin File: `GamepadConfigActivity.kt`
```kotlin
package com.example.gamemapper

import android.content.Context
import android.os.Bundle
import android.view.InputDevice
import android.view.KeyEvent
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Button
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class GamepadConfigActivity : AppCompatActivity() {
    private lateinit var assignButton: Button
    private lateinit var gamepadListView: RecyclerView
    private lateinit var adapter: GamepadButtonAdapter
    private var awaitingKey = false
    private var currentX = 500f
    private var currentY = 500f
    
    // Список назначенных кнопок
    private val assignedButtons = mutableMapOf<Int, Pair<Float, Float>>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_gamepad_config)
        
        assignButton = findViewById(R.id.assignButton)
        gamepadListView = findViewById(R.id.gamepadListView)
        
        // Загружаем текущие назначения кнопок из сервиса
        loadCurrentMappings()
        
        adapter = GamepadButtonAdapter(assignedButtons) { keyCode ->
            // Удаляем назначение
            assignedButtons.remove(keyCode)
            updateServiceMappings()
            adapter.notifyDataSetChanged()
        }
        
        gamepadListView.adapter = adapter
        gamepadListView.layoutManager = LinearLayoutManager(this)
        
        assignButton.setOnClickListener {
            showPositionDialog()
        }
    }
    
    private fun showPositionDialog() {
        val dialogView = layoutInflater.inflate(R.layout.dialog_position_input, null)
        val xInput = dialogView.findViewById<TextView>(R.id.xPositionInput)
        val yInput = dialogView.findViewById<TextView>(R.id.yPositionInput)
        
        xInput.text = currentX.toString()
        yInput.text = currentY.toString()
        
        android.app.AlertDialog.Builder(this)
            .setTitle(R.string.set_position)
            .setView(dialogView)
            .setPositiveButton(R.string.ok) { _, _ ->
                try {
                    currentX = xInput.text.toString().toFloatOrNull() ?: 500f
                    currentY = yInput.text.toString().toFloatOrNull() ?: 500f
                    
                    awaitingKey = true
                    Toast.makeText(this, R.string.press_gamepad_button, Toast.LENGTH_LONG).show()
                } catch (e: Exception) {
                    Toast.makeText(this, R.string.invalid_position, Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }

    override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
        if (awaitingKey && event?.source?.and(InputDevice.SOURCE_GAMEPAD) == InputDevice.SOURCE_GAMEPAD) {
            assignedButtons[keyCode] = Pair(currentX, currentY)
            updateServiceMappings()
            adapter.notifyDataSetChanged()
            
            Toast.makeText(
                this, 
                getString(R.string.button_assigned, KeyEvent.keyCodeToString(keyCode)), 
                Toast.LENGTH_SHORT
            ).show()
            
            awaitingKey = false
            return true
        }
        return super.onKeyDown(keyCode, event)
    }
    
    private fun loadCurrentMappings() {
        val service = getSystemService(Context.ACCESSIBILITY_SERVICE) as? MappingService
        service?.keyMappings?.forEach { (keyCode, position) ->
            // Добавляем только кнопки геймпада
            if (keyCode >= KeyEvent.KEYCODE_BUTTON_A && keyCode <= KeyEvent.KEYCODE_BUTTON_MODE) {
                assignedButtons[keyCode] = position
            }
        }
    }
    
    private fun updateServiceMappings() {
        val service = getSystemService(Context.ACCESSIBILITY_SERVICE) as? MappingService
        service?.let {
            // Удаляем старые назначения кнопок геймпада
            val keysToRemove = it.keyMappings.keys.filter { keyCode -> 
                keyCode >= KeyEvent.KEYCODE_BUTTON_A && keyCode <= KeyEvent.KEYCODE_BUTTON_MODE 
            }
            keysToRemove.forEach { keyCode -> it.keyMappings.remove(keyCode) }
            
            // Добавляем новые назначения
            it.keyMappings.putAll(assignedButtons)
            
            // Обновляем оверлеи
            it.createOverlayButtons()
        }
    }
    
    class GamepadButtonAdapter(
        private val buttons: MutableMap<Int, Pair<Float, Float>>,
        private val onDeleteClick: (Int) -> Unit
    ) : RecyclerView.Adapter<GamepadButtonAdapter.ViewHolder>() {
        
        class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
            val buttonNameText: TextView = view.findViewById(R.id.buttonNameText)
            val positionText: TextView = view.findViewById(R.id.positionText)
            val deleteButton: Button = view.findViewById(R.id.deleteButton)
        }
        
        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val view = LayoutInflater.from(parent.context)
                .inflate(R.layout.item_gamepad_button, parent, false)
            return ViewHolder(view)
        }
        
        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            val keyCode = buttons.keys.elementAt(position)
            val (x, y) = buttons[keyCode]!!
            
            holder.buttonNameText.text = KeyEvent.keyCodeToString(keyCode).replace("KEYCODE_", "")
            holder.positionText.text = "X: $x, Y: $y"
            
            holder.deleteButton.setOnClickListener {
                onDeleteClick(keyCode)
            }
        }
        
        override fun getItemCount() = buttons.size
    }
}

```

## kotlin File: `GameProfile.kt`
```kotlin
package com.example.gamemapper

import android.view.KeyEvent
import java.util.UUID

data class GameProfile(
    val id: String = UUID.randomUUID().toString(),
    var name: String,
    var packageName: String = "",
    val keyMappings: MutableMap<Int, Pair<Float, Float>> = mutableMapOf(),
    val gestureMappings: MutableList<GestureMapping> = mutableListOf(),
    val buttonSettings: MutableMap<Int, ButtonSettings> = mutableMapOf()
)

data class ButtonSettings(
    val shape: Int = CustomOverlayButton.SHAPE_CIRCLE,
    val color: Int = android.graphics.Color.parseColor("#80FFFFFF"),
    val size: Float = 120f,
    val alpha: Int = 128,
    val borderColor: Int = android.graphics.Color.BLACK,
    val borderWidth: Float = 0f
)

data class GestureMapping(
    val id: String = UUID.randomUUID().toString(),
    val gestureType: GestureType,
    val keyCode: Int,
    val startX: Float,
    val startY: Float,
    val endX: Float? = null,
    val endY: Float? = null,
    val duration: Long = 0,
    val tapCount: Int = 1,
    val radius: Float = 0f
)

enum class GestureType {
    TAP, LONG_PRESS, SWIPE, MULTI_TAP, CIRCLE
}

object ProfileManager {
    private val profiles = mutableMapOf<String, GameProfile>()
    
    init {
        // Создаем профиль по умолчанию
        val defaultProfile = GameProfile(
            name = "Default Profile"
        ).apply {
            keyMappings[KeyEvent.KEYCODE_W] = Pair(500f, 300f)
            keyMappings[KeyEvent.KEYCODE_A] = Pair(200f, 500f)
            keyMappings[KeyEvent.KEYCODE_S] = Pair(500f, 700f)
            keyMappings[KeyEvent.KEYCODE_D] = Pair(800f, 500f)
            keyMappings[KeyEvent.KEYCODE_SPACE] = Pair(900f, 800f)
            keyMappings[KeyEvent.KEYCODE_E] = Pair(1000f, 300f)
            keyMappings[KeyEvent.KEYCODE_R] = Pair(1100f, 400f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_A] = Pair(900f, 700f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_B] = Pair(1000f, 600f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_X] = Pair(800f, 600f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_Y] = Pair(900f, 500f)
        }
        
        profiles[defaultProfile.id] = defaultProfile
    }
    
    fun saveProfile(profile: GameProfile) {
        profiles[profile.id] = profile
        PersistenceManager.saveProfiles(profiles.values.toList())
    }
    
    fun getProfile(id: String): GameProfile? {
        return profiles[id]
    }
    
    fun getAllProfiles(): List<GameProfile> {
        return profiles.values.toList()
    }
    
    fun deleteProfile(id: String) {
        profiles.remove(id)
        PersistenceManager.saveProfiles(profiles.values.toList())
    }
    
    fun getProfileForPackage(packageName: String): GameProfile? {
        return profiles.values.find { it.packageName == packageName }
    }
    
    fun loadProfiles() {
        val loadedProfiles = PersistenceManager.loadProfiles()
        profiles.clear()
        loadedProfiles.forEach { profiles[it.id] = it }
        
        // Если нет профилей, создаем профиль по умолчанию
        if (profiles.isEmpty()) {
            val defaultProfile = GameProfile(
                name = "Default Profile"
            ).apply {
                keyMappings[KeyEvent.KEYCODE_W] = Pair(500f, 300f)
                keyMappings[KeyEvent.KEYCODE_A] = Pair(200f, 500f)
                keyMappings[KeyEvent.KEYCODE_S] = Pair(500f, 700f)
                keyMappings[KeyEvent.KEYCODE_D] = Pair(800f, 500f)
                keyMappings[KeyEvent.KEYCODE_SPACE] = Pair(900f, 800f)
                keyMappings[KeyEvent.KEYCODE_E] = Pair(1000f, 300f)
                keyMappings[KeyEvent.KEYCODE_R] = Pair(1100f, 400f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_A] = Pair(900f, 700f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_B] = Pair(1000f, 600f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_X] = Pair(800f, 600f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_Y] = Pair(900f, 500f)
            }
            
            profiles[defaultProfile.id] = defaultProfile
            PersistenceManager.saveProfiles(profiles.values.toList())
        }
    }
}

```

## kotlin File: `GestureConfigActivity.kt`
```kotlin
package com.example.gamemapper

import android.content.Context
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.AdapterView
import android.widget.ArrayAdapter
import android.widget.Button
import android.widget.EditText
import android.widget.Spinner
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class GestureConfigActivity : AppCompatActivity() {
    private lateinit var gesturesRecyclerView: RecyclerView
    private lateinit var addGestureButton: Button
    private lateinit var adapter: GestureAdapter
    private lateinit var currentProfile: GameProfile
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_gesture_config)
        
        gesturesRecyclerView = findViewById(R.id.gesturesRecyclerView)
        addGestureButton = findViewById(R.id.addGestureButton)
        
        // Получаем текущий профиль
        val profileId = intent.getStringExtra("profile_id") ?: return finish()
        currentProfile = ProfileManager.getProfile(profileId) ?: return finish()
        
        title = getString(R.string.gesture_config_for, currentProfile.name)
        
        adapter = GestureAdapter(
            currentProfile.gestureMappings,
            onGestureEdit = { gesture ->
                showGestureDialog(gesture)
            },
            onGestureDelete = { gesture ->
                currentProfile.gestureMappings.remove(gesture)
                ProfileManager.saveProfile(currentProfile)
                adapter.notifyDataSetChanged()
            }
        )
        
        gesturesRecyclerView.adapter = adapter
        gesturesRecyclerView.layoutManager = LinearLayoutManager(this)
        
        addGestureButton.setOnClickListener {
            // Создаем новый жест по умолчанию
            val newGesture = GestureMapping(
                gestureType = GestureType.TAP,
                keyCode = android.view.KeyEvent.KEYCODE_UNKNOWN,
                startX = 500f,
                startY = 500f
            )
            showGestureDialog(newGesture, isNew = true)
        }
    }
    
    private fun showGestureDialog(gesture: GestureMapping, isNew: Boolean = false) {
        val dialogView = layoutInflater.inflate(R.layout.dialog_gesture_config, null)
        
        val gestureTypeSpinner = dialogView.findViewById<Spinner>(R.id.gestureTypeSpinner)
        val keyCodeSpinner = dialogView.findViewById<Spinner>(R.id.keyCodeSpinner)
        val startXEdit = dialogView.findViewById<EditText>(R.id.startXEdit)
        val startYEdit = dialogView.findViewById<EditText>(R.id.startYEdit)
        val endXEdit = dialogView.findViewById<EditText>(R.id.endXEdit)
        val endYEdit = dialogView.findViewById<EditText>(R.id.endYEdit)
        val durationEdit = dialogView.findViewById<EditText>(R.id.durationEdit)
        val tapCountEdit = dialogView.findViewById<EditText>(R.id.tapCountEdit)
        val radiusEdit = dialogView.findViewById<EditText>(R.id.radiusEdit)
        
        val endCoordinatesLayout = dialogView.findViewById<View>(R.id.endCoordinatesLayout)
        val durationLayout = dialogView.findViewById<View>(R.id.durationLayout)
        val tapCountLayout = dialogView.findViewById<View>(R.id.tapCountLayout)
        val radiusLayout = dialogView.findViewById<View>(R.id.radiusLayout)
        
        // Настраиваем спиннер типов жестов
        val gestureTypes = GestureType.values().map { it.name }
        val gestureAdapter = ArrayAdapter(this, android.R.layout.simple_spinner_item, gestureTypes)
        gestureAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
        gestureTypeSpinner.adapter = gestureAdapter
        gestureTypeSpinner.setSelection(gestureTypes.indexOf(gesture.gestureType.name))
        
        // Настраиваем спиннер кодов клавиш
        val keyCodes = arrayOf(
            android.view.KeyEvent.KEYCODE_A,
            android.view.KeyEvent.KEYCODE_B,
            android.view.KeyEvent.KEYCODE_C,
            // Добавьте больше кодов клавиш по необходимости
            android.view.KeyEvent.KEYCODE_BUTTON_A,
            android.view.KeyEvent.KEYCODE_BUTTON_B,
            android.view.KeyEvent.KEYCODE_BUTTON_X,
            android.view.KeyEvent.KEYCODE_BUTTON_Y
        )
        
        val keyCodeNames = keyCodes.map { 
            android.view.KeyEvent.keyCodeToString(it).replace("KEYCODE_", "") 
        }
        
        val keyCodeAdapter = ArrayAdapter(this, android.R.layout.simple_spinner_item, keyCodeNames)
        keyCodeAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
        keyCodeSpinner.adapter = keyCodeAdapter
        
        val keyCodeIndex = keyCodes.indexOf(gesture.keyCode)
        if (keyCodeIndex >= 0) {
            keyCodeSpinner.setSelection(keyCodeIndex)
        }
        
        // Заполняем поля текущими значениями
        startXEdit.setText(gesture.startX.toString())
        startYEdit.setText(gesture.startY.toString())
        gesture.endX?.let { endXEdit.setText(it.toString()) }
        gesture.endY?.let { endYEdit.setText(it.toString()) }
        durationEdit.setText(gesture.duration.toString())
        tapCountEdit.setText(gesture.tapCount.toString())
        radiusEdit.setText(gesture.radius.toString())
        
        // Настраиваем видимость полей в зависимости от типа жеста
        gestureTypeSpinner.onItemSelectedListener = object : AdapterView.OnItemSelectedListener {
            override fun onItemSelected(parent: AdapterView<*>, view: View?, position: Int, id: Long) {
                when (GestureType.valueOf(gestureTypes[position])) {
                    GestureType.TAP -> {
                        endCoordinatesLayout.visibility = View.GONE
                        durationLayout.visibility = View.GONE
                        tapCountLayout.visibility = View.GONE
                        radiusLayout.visibility = View.GONE
                    }
                    GestureType.LONG_PRESS -> {
                        endCoordinatesLayout.visibility = View.GONE
                        durationLayout.visibility = View.VISIBLE
                        tapCountLayout.visibility = View.GONE
                        radiusLayout.visibility = View.GONE
                    }
                    GestureType.SWIPE -> {
                        endCoordinatesLayout.visibility = View.VISIBLE
                        durationLayout.visibility = View.VISIBLE
                        tapCountLayout.visibility = View.GONE
                        radiusLayout.visibility = View.GONE
                    }
                    GestureType.MULTI_TAP -> {
                        endCoordinatesLayout.visibility = View.GONE
                        durationLayout.visibility = View.GONE
                        tapCountLayout.visibility = View.VISIBLE
                        radiusLayout.visibility = View.GONE
                    }
                    GestureType.CIRCLE -> {
                        endCoordinatesLayout.visibility = View.GONE
                        durationLayout.visibility = View.VISIBLE
                        tapCountLayout.visibility = View.GONE
                        radiusLayout.visibility = View.VISIBLE
                    }
                }
            }
            
            override fun onNothingSelected(parent: AdapterView<*>) {}
        }
        
        // Вызываем onItemSelected вручную для установки начального состояния
        gestureTypeSpinner.onItemSelectedListener.onItemSelected(
            gestureTypeSpinner, null, gestureTypeSpinner.selectedItemPosition, 0
        )
        
        AlertDialog.Builder(this)
            .setTitle(if (isNew) R.string.add_gesture else R.string.edit_gesture)
            .setView(dialogView)
            .setPositiveButton(R.string.save) { _, _ ->
                try {
                    // Получаем значения из полей
                    val gestureType = GestureType.valueOf(gestureTypes[gestureTypeSpinner.selectedItemPosition])
                    val keyCode = keyCodes[keyCodeSpinner.selectedItemPosition]
                    val startX = startXEdit.text.toString().toFloatOrNull() ?: 0f
                    val startY = startYEdit.text.toString().toFloatOrNull() ?: 0f
                    
                    // Опциональные поля в зависимости от типа жеста
                    val endX = if (endCoordinatesLayout.visibility == View.VISIBLE) 
                        endXEdit.text.toString().toFloatOrNull() else null
                    val endY = if (endCoordinatesLayout.visibility == View.VISIBLE) 
                        endYEdit.text.toString().toFloatOrNull() else null
                    val duration = if (durationLayout.visibility == View.VISIBLE) 
                        durationEdit.text.toString().toLongOrNull() ?: 500L else 0L
                    val tapCount = if (tapCountLayout.visibility == View.VISIBLE) 
                        tapCountEdit.text.toString().toIntOrNull() ?: 1 else 1
                    val radius = if (radiusLayout.visibility == View.VISIBLE) 
                        radiusEdit.text.toString().toFloatOrNull() ?: 100f else 0f
                    
                    // Создаем обновленный жест
                    val updatedGesture = GestureMapping(
                        id = gesture.id,
                        gestureType = gestureType,
                        keyCode = keyCode,
                        startX = startX,
                        startY = startY,
                        endX = endX,
                        endY = endY,
                        duration = duration,
                        tapCount = tapCount,
                        radius = radius
                    )
                    
                    // Добавляем или обновляем жест в профиле
                    if (isNew) {
                        currentProfile.gestureMappings.add(updatedGesture)
                    } else {
                        val index = currentProfile.gestureMappings.indexOfFirst { it.id == gesture.id }
                        if (index >= 0) {
                            currentProfile.gestureMappings[index] = updatedGesture
                        }
                    }
                    
                    // Сохраняем профиль
                    ProfileManager.saveProfile(currentProfile)
                    adapter.notifyDataSetChanged()
                    
                    Toast.makeText(
                        this, 
                        if (isNew) R.string.gesture_added else R.string.gesture_updated, 
                        Toast.LENGTH_SHORT
                    ).show()
                } catch (e: Exception) {
                    Toast.makeText(this, R.string.gesture_config_error, Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    class GestureAdapter(
        val gestures: MutableList<GestureMapping>,
        private val onGestureEdit: (GestureMapping) -> Unit,
        private val onGestureDelete: (GestureMapping) -> Unit
    ) : RecyclerView.Adapter<GestureAdapter.ViewHolder>() {
        
        class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
            val gestureTypeText: TextView = view.findViewById(R.id.gestureTypeText)
            val keyCodeText: TextView = view.findViewById(R.id.keyCodeText)
            val gestureDetailsText: TextView = view.findViewById(R.id.gestureDetailsText)
            val editButton: Button = view.findViewById(R.id.editButton)
            val deleteButton: Button = view.findViewById(R.id.deleteButton)
        }
        
        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val view = LayoutInflater.from(parent.context)
                .inflate(R.layout.item_gesture, parent, false)
            return ViewHolder(view)
        }
        
        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            val gesture = gestures[position]
            val context = holder.itemView.context
            
            holder.gestureTypeText.text = gesture.gestureType.name
            holder.keyCodeText.text = android.view.KeyEvent.keyCodeToString(gesture.keyCode).replace("KEYCODE_", "")
            
            // Формируем детальное описание жеста в зависимости от его типа
            val details = StringBuilder()
            details.append("X: ${gesture.startX}, Y: ${gesture.startY}")
            
            when (gesture.gestureType) {
                GestureType.SWIPE -> {
                    gesture.endX?.let { endX ->
                        gesture.endY?.let { endY ->
                            details.append(" → X: $endX, Y: $endY")
                        }
                    }
                    details.append(", ${context.getString(R.string.duration)}: ${gesture.duration}ms")
                }
                GestureType.LONG_PRESS -> {
                    details.append(", ${context.getString(R.string.duration)}: ${gesture.duration}ms")
                }
                GestureType.MULTI_TAP -> {
                    details.append(", ${context.getString(R.string.tap_count)}: ${gesture.tapCount}")
                }
                GestureType.CIRCLE -> {
                    details.append(", ${context.getString(R.string.radius)}: ${gesture.radius}")
                    details.append(", ${context.getString(R.string.duration)}: ${gesture.duration}ms")
                }
                else -> {}
            }
            
            holder.gestureDetailsText.text = details.toString()
            
            holder.editButton.setOnClickListener {
                onGestureEdit(gesture)
            }
            
            holder.deleteButton.setOnClickListener {
                onGestureDelete(gesture)
            }
        }
        
        override fun getItemCount() = gestures.size
    }
}

```

## kotlin File: `KeyMappingAdapter.kt`
```kotlin
package com.example.gamemapper

import android.text.Editable
import android.text.TextWatcher
import android.view.KeyEvent
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class KeyMappingAdapter(
    private val mappings: MutableMap<Int, Pair<Float, Float>>,
    private val onPositionChanged: (Int, Float, Float) -> Unit
) : RecyclerView.Adapter<KeyMappingAdapter.ViewHolder>() {

    class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val keyText: TextView = view.findViewById(android.R.id.text1)
        val xEdit: EditText = view.findViewById(android.R.id.edit)
        val yEdit: EditText = view.findViewById(android.R.id.custom)
        val deleteButton: Button = view.findViewById(android.R.id.button1)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_key_mapping, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val keyCode = mappings.keys.elementAt(position)
        val (x, y) = mappings[keyCode]!!
        holder.keyText.text = KeyEvent.keyCodeToString(keyCode).replace("KEYCODE_", "")
        holder.xEdit.setText(x.toString())
        holder.yEdit.setText(y.toString())

        holder.xEdit.addTextChangedListener(object : TextWatcher {
            override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {}
            override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {}
            override fun afterTextChanged(s: Editable?) {
                s?.toString()?.toFloatOrNull()?.let { onPositionChanged(keyCode, it, y) }
            }
        })

        holder.yEdit.addTextChangedListener(object : TextWatcher {
            override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {}
            override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {}
            override fun afterTextChanged(s: Editable?) {
                s?.toString()?.toFloatOrNull()?.let { onPositionChanged(keyCode, x, it) }
            }
        })

        holder.deleteButton.setOnClickListener {
            mappings.remove(keyCode)
            notifyItemRemoved(position)
        }
    }

    override fun getItemCount() = mappings.size
}
```

## kotlin File: `RecorderManager.kt`
```kotlin
package com.example.gamemapper

import android.content.Context
import android.net.Uri
import android.util.Log
import android.widget.Toast
import java.io.BufferedReader
import java.io.InputStreamReader
import java.io.OutputStreamWriter
import java.util.concurrent.atomic.AtomicBoolean

object RecorderManager {
    private const val TAG = "RecorderManager"
    private val isRecording = AtomicBoolean(false)
    private val recordedActions = mutableListOf<TouchAction>()
    
    data class TouchAction(
        val x: Float,
        val y: Float,
        val type: ActionType,
        val timestamp: Long = System.currentTimeMillis()
    )
    
    enum class ActionType {
        TAP, SWIPE_START, SWIPE_END, LONG_PRESS
    }
    
    fun startRecording() {
        if (!isRecording.getAndSet(true)) {
            recordedActions.clear()
            Log.d(TAG, "Recording started")
        }
    }
    
    fun stopRecording() {
        if (isRecording.getAndSet(false)) {
            Log.d(TAG, "Recording stopped with ${recordedActions.size} actions")
        }
    }
    
    fun isRecording(): Boolean = isRecording.get()
    
    fun recordAction(x: Float, y: Float, type: ActionType) {
        if (isRecording.get()) {
            recordedActions.add(TouchAction(x, y, type))
            Log.d(TAG, "Recorded action: $type at ($x, $y)")
        }
    }
    
    fun getRecordedActions(): List<TouchAction> = recordedActions.toList()
    
    fun saveRecording(context: Context, uri: Uri): Boolean {
        try {
            context.contentResolver.openOutputStream(uri)?.use { outputStream ->
                OutputStreamWriter(outputStream).use { writer ->
                    recordedActions.forEach { action ->
                        writer.write("${action.x},${action.y},${action.type},${action.timestamp}\n")
                    }
                }
            }
            Log.d(TAG, "Saved ${recordedActions.size} actions to $uri")
            return true
        } catch (e: Exception) {
            Log.e(TAG, "Error saving recording: ${e.message}", e)
            return false
        }
    }
    
    fun loadRecording(context: Context, uri: Uri): Boolean {
        try {
            val actions = mutableListOf<TouchAction>()
            
            context.contentResolver.openInputStream(uri)?.use { inputStream ->
                BufferedReader(InputStreamReader(inputStream)).use { reader ->
                    reader.forEachLine { line ->
                        val parts = line.split(",")
                        if (parts.size >= 4) {
                            try {
                                val x = parts[0].toFloat()
                                val y = parts[1].toFloat()
                                val type = ActionType.valueOf(parts[2])
                                val timestamp = parts[3].toLong()
                                actions.add(TouchAction(x, y, type, timestamp))
                            } catch (e: Exception) {
                                Log.e(TAG, "Error parsing line: $line", e)
                            }
                        }
                    }
                }
            }
            
            if (actions.isNotEmpty()) {
                recordedActions.clear()
                recordedActions.addAll(actions)
                Log.d(TAG, "Loaded ${actions.size} actions from $uri")
                return true
            }
            return false
        } catch (e: Exception) {
            Log.e(TAG, "Error loading recording: ${e.message}", e)
            return false
        }
    }
    
    fun playRecording(service: MappingService, context: Context) {
        if (recordedActions.isEmpty()) {
            return
        }
        
        service.scope.launch {
            try {
                val startTime = System.currentTimeMillis()
                val firstActionTime = recordedActions.first().timestamp
                
                for (action in recordedActions) {
                    val elapsedTime = action.timestamp - firstActionTime
                    val waitTime = elapsedTime - (System.currentTimeMillis() - startTime)
                    
                    if (waitTime > 0) {
                        delay(waitTime)
                    }
                    
                    when (action.type) {
                        ActionType.TAP -> {
                            service.performVirtualTouch(action.x, action.y)
                        }
                        ActionType.SWIPE_START -> {
                            // Находим соответствующий SWIPE_END
                            val endAction = recordedActions.find { 
                                it.type == ActionType.SWIPE_END && it.timestamp > action.timestamp 
                            }
                            
                            if (endAction != null) {
                                service.simulateDrag(action.x, action.y, endAction.x, endAction.y)
                            }
                        }
                        ActionType.LONG_PRESS -> {
                            service.performLongPress(action.x, action.y, 1000)
                        }
                        else -> {}
                    }
                }
                
                Toast.makeText(context, R.string.playback_completed, Toast.LENGTH_SHORT).show()
            } catch (e: Exception) {
                Log.e(TAG, "Error during playback: ${e.message}", e)
                Toast.makeText(context, R.string.playback_error, Toast.LENGTH_SHORT).show()
            }
        }
    }
}

```

## kotlin File: `ProfileManagementActivity.kt`
```kotlin
package com.example.gamemapper

import android.app.AlertDialog
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Button
import android.widget.EditText
import android.widget.ImageButton
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class ProfileManagementActivity : AppCompatActivity() {
    private lateinit var profilesRecyclerView: RecyclerView
    private lateinit var addProfileButton: Button
    private lateinit var adapter: ProfileAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_profile_management)
        
        profilesRecyclerView = findViewById(R.id.profilesRecyclerView)
        addProfileButton = findViewById(R.id.addProfileButton)
        
        // Загружаем профили
        ProfileManager.loadProfiles()
        
        adapter = ProfileAdapter(ProfileManager.getAllProfiles().toMutableList(), 
            onProfileSelected = { profile ->
                // Загружаем выбранный профиль в сервис
                val intent = Intent(this, MappingService::class.java).apply {
                    action = "LOAD_PROFILE"
                    putExtra("profile_id", profile.id)
                }
                startService(intent)
                Toast.makeText(this, getString(R.string.profile_activated, profile.name), Toast.LENGTH_SHORT).show()
            },
            onProfileEdit = { profile ->
                showEditProfileDialog(profile)
            },
            onProfileDelete = { profile ->
                showDeleteProfileDialog(profile)
            },
            onAssignApp = { profile ->
                showAppSelectionDialog(profile)
            }
        )
        
        profilesRecyclerView.adapter = adapter
        profilesRecyclerView.layoutManager = LinearLayoutManager(this)
        
        addProfileButton.setOnClickListener {
            showCreateProfileDialog()
        }
    }
    
    private fun showCreateProfileDialog() {
        val dialogView = layoutInflater.inflate(R.layout.dialog_create_profile, null)
        val nameEditText = dialogView.findViewById<EditText>(R.id.profileNameEditText)
        
        AlertDialog.Builder(this)
            .setTitle(R.string.create_profile)
            .setView(dialogView)
            .setPositiveButton(R.string.create) { _, _ ->
                val name = nameEditText.text.toString().trim()
                if (name.isNotEmpty()) {
                    val newProfile = GameProfile(name = name)
                    
                    // Копируем маппинги из профиля по умолчанию
                    val defaultProfile = ProfileManager.getAllProfiles().firstOrNull()
                    defaultProfile?.let {
                        newProfile.keyMappings.putAll(it.keyMappings)
                    }
                    
                    ProfileManager.saveProfile(newProfile)
                    updateProfilesList()
                    
                    Toast.makeText(this, getString(R.string.profile_created, name), Toast.LENGTH_SHORT).show()
                } else {
                    Toast.makeText(this, R.string.profile_name_required, Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun showEditProfileDialog(profile: GameProfile) {
        val dialogView = layoutInflater.inflate(R.layout.dialog_create_profile, null)
        val nameEditText = dialogView.findViewById<EditText>(R.id.profileNameEditText)
        nameEditText.setText(profile.name)
        
        AlertDialog.Builder(this)
            .setTitle(R.string.edit_profile)
            .setView(dialogView)
            .setPositiveButton(R.string.save) { _, _ ->
                val name = nameEditText.text.toString().trim()
                if (name.isNotEmpty()) {
                    profile.name = name
                    ProfileManager.saveProfile(profile)
                    updateProfilesList()
                    
                    Toast.makeText(this, getString(R.string.profile_updated, name), Toast.LENGTH_SHORT).show()
                } else {
                    Toast.makeText(this, R.string.profile_name_required, Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun showDeleteProfileDialog(profile: GameProfile) {
        AlertDialog.Builder(this)
            .setTitle(R.string.delete_profile)
            .setMessage(getString(R.string.delete_profile_confirmation, profile.name))
            .setPositiveButton(R.string.delete) { _, _ ->
                ProfileManager.deleteProfile(profile.id)
                updateProfilesList()
                
                Toast.makeText(this, getString(R.string.profile_deleted, profile.name), Toast.LENGTH_SHORT).show()
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun showAppSelectionDialog(profile: GameProfile) {
        val packageManager = packageManager
        val installedApps = packageManager.getInstalledApplications(PackageManager.GET_META_DATA)
            .filter { packageManager.getLaunchIntentForPackage(it.packageName) != null }
            .sortedBy { packageManager.getApplicationLabel(it).toString() }
        
        val appNames = installedApps.map { 
            packageManager.getApplicationLabel(it).toString()
        }.toTypedArray()
        
        AlertDialog.Builder(this)
            .setTitle(R.string.select_app)
            .setItems(appNames) { _, which ->
                val selectedApp = installedApps[which]
                profile.packageName = selectedApp.packageName
                ProfileManager.saveProfile(profile)
                updateProfilesList()
                
                Toast.makeText(
                    this, 
                    getString(
                        R.string.app_assigned_to_profile, 
                        packageManager.getApplicationLabel(selectedApp),
                        profile.name
                    ),
                    Toast.LENGTH_SHORT
                ).show()
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun updateProfilesList() {
        adapter.profiles.clear()
        adapter.profiles.addAll(ProfileManager.getAllProfiles())
        adapter.notifyDataSetChanged()
    }
    
    class ProfileAdapter(
        val profiles: MutableList<GameProfile>,
        private val onProfileSelected: (GameProfile) -> Unit,
        private val onProfileEdit: (GameProfile) -> Unit,
        private val onProfileDelete: (GameProfile) -> Unit,
        private val onAssignApp: (GameProfile) -> Unit
    ) : RecyclerView.Adapter<ProfileAdapter.ViewHolder>() {
        
        class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
            val profileNameTextView: TextView = view.findViewById(R.id.profileNameTextView)
            val appNameTextView: TextView = view.findViewById(R.id.appNameTextView)
            val editButton: ImageButton = view.findViewById(R.id.editButton)
            val deleteButton: ImageButton = view.findViewById(R.id.deleteButton)
            val assignAppButton: Button = view.findViewById(R.id.assignAppButton)
        }
        
        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val view = LayoutInflater.from(parent.context)
                .inflate(R.layout.item_profile, parent, false)
            return ViewHolder(view)
        }
        
        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            val profile = profiles[position]
            val context = holder.itemView.context
            
            holder.profileNameTextView.text = profile.name
            
            // Отображаем имя приложения, если оно назначено
            if (profile.packageName.isNotEmpty()) {
                try {
                    val packageManager = context.packageManager
                    val appInfo = packageManager.getApplicationInfo(profile.packageName, 0)
                    val appName = packageManager.getApplicationLabel(appInfo)
                    holder.appNameTextView.text = appName
                } catch (e: Exception) {
                    holder.appNameTextView.text = profile.packageName
                }
            } else {
                holder.appNameTextView.text = context.getString(R.string.no_app_assigned)
            }
            
            // Устанавливаем обработчики событий
            holder.itemView.setOnClickListener {
                onProfileSelected(profile)
            }
            
            holder.editButton.setOnClickListener {
                onProfileEdit(profile)
            }
            
            holder.deleteButton.setOnClickListener {
                onProfileDelete(profile)
            }
            
            holder.assignAppButton.setOnClickListener {
                onAssignApp(profile)
            }
        }
        
        override fun getItemCount() = profiles.size
    }
}

```

## kotlin File: `PersistenceManager.kt`
```kotlin
package com.example.gamemapper

import android.content.Context
import android.util.Log
import com.google.gson.Gson
import com.google.gson.GsonBuilder
import com.google.gson.reflect.TypeToken
import java.io.File

object PersistenceManager {
    private const val TAG = "PersistenceManager"
    private const val PROFILES_FILENAME = "profiles.json"
    private lateinit var appContext: Context
    private val gson: Gson = GsonBuilder().setPrettyPrinting().create()
    
    fun init(context: Context) {
        appContext = context.applicationContext
    }
    
    fun saveProfiles(profiles: List<GameProfile>) {
        try {
            val json = gson.toJson(profiles)
            val file = File(appContext.filesDir, PROFILES_FILENAME)
            file.writeText(json)
            Log.d(TAG, "Saved ${profiles.size} profiles")
        } catch (e: Exception) {
            Log.e(TAG, "Error saving profiles: ${e.message}", e)
        }
    }
    
    fun loadProfiles(): List<GameProfile> {
        try {
            val file = File(appContext.filesDir, PROFILES_FILENAME)
            if (!file.exists()) {
                Log.d(TAG, "Profiles file not found, returning empty list")
                return emptyList()
            }
            
            val json = file.readText()
            val type = object : TypeToken<List<GameProfile>>() {}.type
            val profiles = gson.fromJson<List<GameProfile>>(json, type)
            Log.d(TAG, "Loaded ${profiles.size} profiles")
            return profiles
        } catch (e: Exception) {
            Log.e(TAG, "Error loading profiles: ${e.message}", e)
            return emptyList()
        }
    }
}

```

## kotlin File: `MappingService.kt`
```kotlin
package com.example.gamemapper

import android.accessibilityservice.AccessibilityService
import android.accessibilityservice.AccessibilityServiceInfo
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.content.Context
import android.content.Intent
import android.graphics.Color
import android.graphics.PixelFormat
import android.graphics.Rect
import android.hardware.input.InputManager
import android.os.Build
import android.util.Log
import android.view.Gravity
import android.view.InputDevice
import android.view.InputEvent
import android.view.KeyEvent
import android.view.MotionEvent
import android.view.View
import android.view.WindowManager
import android.view.accessibility.AccessibilityEvent
import android.widget.Toast
import androidx.annotation.RequiresApi
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.NotificationCompat
import kotlinx.coroutines.*
import java.lang.reflect.Method

class MappingService : AccessibilityService() {
    private lateinit var windowManager: WindowManager
    val overlayButtons = mutableMapOf<Int, View>()
    private var mouseLookArea: View? = null
    val keyMappings = mutableMapOf<Int, Pair<Float, Float>>()
    private lateinit var inputManager: InputManager
    private var injectInputEventMethod: Method? = null
    val scope = CoroutineScope(Dispatchers.Main + Job())
    
    private var currentProfile: GameProfile? = null
    private var isServiceActive = false
    
    companion object {
        private const val TAG = "MappingService"
        private const val NOTIFICATION_CHANNEL_ID = "GameMapperChannel"
        private const val NOTIFICATION_ID = 1
    }

    override fun onServiceConnected() {
        try {
            val info = AccessibilityServiceInfo().apply {
                eventTypes = AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED or
                          AccessibilityEvent.TYPE_VIEW_CLICKED or
                         AccessibilityEvent.TYPE_VIEW_FOCUSED
                feedbackType = AccessibilityServiceInfo.FEEDBACK_GENERIC
                flags = AccessibilityServiceInfo.FLAG_REQUEST_FILTER_KEY_EVENTS or
                    AccessibilityServiceInfo.FLAG_REQUEST_TOUCH_EXPLORATION_MODE
                notificationTimeout = 100
            }
            serviceInfo = info
            windowManager = getSystemService(Context.WINDOW_SERVICE) as WindowManager
            inputManager = getSystemService(Context.INPUT_SERVICE) as InputManager
            
            try {
                injectInputEventMethod = InputManager::class.java.getDeclaredMethod(
                    "injectInputEvent", InputEvent::class.java, Int::class.java
                ).apply { isAccessible = true }
            } catch (e: Exception) {
                Log.e(TAG, "Failed to get injectInputEvent method: ${e.message}", e)
            }
            setupDefaultKeyMappings()
            loadOverlayPositions()
            createOverlayButtons()
            
            createNotificationChannel()
            startForeground(NOTIFICATION_ID, createNotification())
            
            isServiceActive = true
            Log.i(TAG, "Service connected successfully")
            Toast.makeText(this, R.string.service_started, Toast.LENGTH_SHORT).show()
        } catch (e: Exception) {
            Log.e(TAG, "Failed to connect service: ${e.message}", e)
            Toast.makeText(this, R.string.service_start_error, Toast.LENGTH_LONG).show()
        }
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        if (intent?.action == "STOP_SERVICE") {
            stopSelf()
        } else if (intent?.action == "LOAD_PROFILE") {
            val profileId = intent.getStringExtra("profile_id")
            profileId?.let { loadProfile(it) }
        }
        return START_STICKY
    }
    
    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                NOTIFICATION_CHANNEL_ID,
                getString(R.string.notification_channel_name),
                NotificationManager.IMPORTANCE_LOW
            ).apply {
                description = getString(R.string.notification_channel_description)
            }
            
            val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            notificationManager.createNotificationChannel(channel)
        }
    }
    
    private fun createNotification() = NotificationCompat.Builder(this, NOTIFICATION_CHANNEL_ID)
        .setSmallIcon(R.drawable.ic_gamepad)
        .setContentTitle(getString(R.string.notification_title))
        .setContentText(getString(R.string.notification_text))
        .setPriority(NotificationCompat.PRIORITY_LOW)
        .setOngoing(true)
        .addAction(
            R.drawable.ic_stop,
            getString(R.string.stop),
            PendingIntent.getService(
                this,
                0,
                Intent(this, MappingService::class.java).apply { action = "STOP_SERVICE" },
                PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
            )
        )
        .build()

    override fun onAccessibilityEvent(event: AccessibilityEvent) {
        if (!isServiceActive) return
        
        try {
            when (event.eventType) {
                AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED -> {
                    // Автоматическое определение приложения и загрузка профиля
                    event.packageName?.toString()?.let { packageName ->
                        val profile = ProfileManager.getProfileForPackage(packageName)
                        profile?.let { loadProfile(it.id) }
                    }
                }
                
                AccessibilityEvent.TYPE_VIEW_CLICKED -> {
                    // Обработка нажатий на элементы интерфейса
                    event.source?.let { node ->
                        val rect = Rect()
                        node.getBoundsInScreen(rect)
                        Log.d(TAG, "View clicked at (${rect.centerX()}, ${rect.centerY()})")
                    }
                }
            }
        } catch (e: Exception) {
            Log.e(TAG, "Error in onAccessibilityEvent: ${e.message}", e)
        }
    }

    override fun onInterrupt() {
        Log.w(TAG, "Service interrupted")
    }

    @RequiresApi(Build.VERSION_CODES.O)
    override fun onKeyEvent(event: KeyEvent): Boolean {
        if (!isServiceActive) return false
        
        try {
            val source = event.device?.sources ?: return false
            
            // Обработка событий геймпада
            if (source and InputDevice.SOURCE_GAMEPAD == InputDevice.SOURCE_GAMEPAD) {
                val keyCode = event.keyCode
                
                if (event.action == KeyEvent.ACTION_DOWN) {
                    // Проверяем наличие жестов для этой кнопки
                    processGesture(keyCode)
                    
                    // Если есть обычный маппинг, выполняем его
                    val mapping = keyMappings[keyCode]
                    mapping?.let {
                        scope.launch {
                            performVirtualTouch(it.first, it.second)
                            highlightButton(keyCode)
                        }
                        Log.d(TAG, "Gamepad key event processed: $keyCode -> (${it.first}, ${it.second})")
                        return true
                    }
                }
            }
            
            // Обработка событий клавиатуры
            if (source and InputDevice.SOURCE_KEYBOARD == InputDevice.SOURCE_KEYBOARD) {
                val keyCode = event.keyCode
                
                if (event.action == KeyEvent.ACTION_DOWN) {
                    // Проверяем наличие жестов для этой клавиши
                    processGesture(keyCode)
                    
                    // Если есть обычный маппинг, выполняем его
                    val mapping = keyMappings[keyCode]
                    mapping?.let {
                        scope.launch {
                            performVirtualTouch(it.first, it.second)
                            highlightButton(keyCode)
                        }
                        Log.d(TAG, "Keyboard key event processed: $keyCode -> (${it.first}, ${it.second})")
                        return true
                    }
                }
            }
            
            return false
        } catch (e: Exception) {
            Log.e(TAG, "Error processing key event: ${e.message}", e)
            return false
        }
    }

    private fun highlightButton(keyCode: Int) {
        overlayButtons[keyCode]?.let { button ->
            button.setBackgroundColor(Color.parseColor("#80FF0000"))
            scope.launch {
                delay(100)
                button.setBackgroundColor(Color.parseColor("#80FFFFFF"))
            }
        }
    }

    @RequiresApi(Build.VERSION_CODES.O)
    fun processMotionEvent(event: MotionEvent): Boolean {
        if (!isServiceActive) return false
        
        try {
            // Обработка событий геймпада (аналоговые стики)
            if (event.source and InputDevice.SOURCE_GAMEPAD == InputDevice.SOURCE_GAMEPAD) {
                val axisX = event.getAxisValue(MotionEvent.AXIS_X)
                val axisY = event.getAxisValue(MotionEvent.AXIS_Y)
                val axisRX = event.getAxisValue(MotionEvent.AXIS_RX)
                val axisRY = event.getAxisValue(MotionEvent.AXIS_RY)
                
                // Обработка левого стика
                if (Math.abs(axisX) > 0.3f || Math.abs(axisY) > 0.3f) {
                    scope.launch {
                        val centerX = 500f
                        val centerY = 500f
                        val sensitivity = 150f
                        simulateDrag(
                            centerX, 
                            centerY, 
                            centerX + axisX * sensitivity, 
                            centerY + axisY * sensitivity
                        )
                    }
                    return true
                }
                
                // Обработка правого стика
                if (Math.abs(axisRX) > 0.3f || Math.abs(axisRY) > 0.3f) {
                    scope.launch {
                        val centerX = 800f
                        val centerY = 500f
                        val sensitivity = 150f
                        simulateDrag(
                            centerX, 
                            centerY, 
                            centerX + axisRX * sensitivity, 
                            centerY + axisRY * sensitivity
                        )
                    }
                    return true
                }
            }
            
            // Обработка событий мыши
            if (event.source and InputDevice.SOURCE_MOUSE == InputDevice.SOURCE_MOUSE) {
                // Обработка области для управления взглядом
                if (event.action == MotionEvent.ACTION_HOVER_MOVE && mouseLookArea != null) {
                    val x = event.rawX
                    val y = event.rawY
                    val area = mouseLookArea!!
                    val location = IntArray(2)
                    area.getLocationOnScreen(location)
                    val areaX = location[0]
                    val areaY = location[1]
                    val areaWidth = area.width
                    val areaHeight = area.height
                    
                    if (x >= areaX && x <= areaX + areaWidth && y >= areaY && y <= areaY + areaHeight) {
                        val relativeX = x - areaX
                        val relativeY = y - areaY
                        val centerX = areaWidth / 2f
                        val centerY = areaHeight / 2f
                        val deltaX = relativeX - centerX
                        val deltaY = relativeY - centerY
                        val scaleFactor = 0.5f
                        
                        if (Math.abs(deltaX) > 5 || Math.abs(deltaY) > 5) {
                            scope.launch {
                                simulateDrag(
                                    areaX + centerX,
                                    areaY + centerY,
                                    areaX + centerX + deltaX * scaleFactor,
                                    areaY + centerY + deltaY * scaleFactor
                                )
                            }
                            Log.d(TAG, "Mouse look processed: deltaX=$deltaX, deltaY=$deltaY")
                        }
                        return true
                    }
                }
                
                // Обработка кликов мыши
                if (event.buttonState and MotionEvent.BUTTON_PRIMARY != 0 &&
                    event.action == MotionEvent.ACTION_BUTTON_PRESS) {
                    val x = event.rawX
                    val y = event.rawY
                    scope.launch { performVirtualTouch(x, y) }
                    Log.d(TAG, "Mouse click processed at: x=$x, y=$y")
                    return true
                }
            }
            
            return false
        } catch (e: Exception) {
            Log.e(TAG, "Error processing motion event: ${e.message}", e)
            return false
        }
    }

    // Обработка жестов из активного профиля
    private fun processGesture(keyCode: Int) {
        currentProfile?.let { profile ->
            val matchingGestures = profile.gestureMappings.filter { it.keyCode == keyCode }
            if (matchingGestures.isNotEmpty()) {
                // Выполняем все жесты, связанные с этой клавишей
                scope.launch {
                    matchingGestures.forEach { gesture ->
                        when (gesture.gestureType) {
                            GestureType.TAP -> {
                                performVirtualTouch(gesture.startX, gesture.startY)
                                Log.d(TAG, "Processed TAP gesture at (${gesture.startX}, ${gesture.startY})")
                            }
                            GestureType.LONG_PRESS -> {
                                performLongPress(gesture.startX, gesture.startY, gesture.duration)
                                Log.d(TAG, "Processed LONG_PRESS gesture at (${gesture.startX}, ${gesture.startY}) for ${gesture.duration}ms")
                            }
                            GestureType.SWIPE -> {
                                gesture.endX?.let { endX ->
                                    gesture.endY?.let { endY ->
                                        simulateDrag(gesture.startX, gesture.startY, endX, endY)
                                        Log.d(TAG, "Processed SWIPE gesture from (${gesture.startX}, ${gesture.startY}) to ($endX, $endY)")
                                    }
                                }
                            }
                            GestureType.MULTI_TAP -> {
                                performMultiTap(gesture.startX, gesture.startY, gesture.tapCount)
                                Log.d(TAG, "Processed MULTI_TAP gesture at (${gesture.startX}, ${gesture.startY}), count: ${gesture.tapCount}")
                            }
                            GestureType.CIRCLE -> {
                                performCircularMotion(gesture.startX, gesture.startY, gesture.radius, gesture.duration)
                                Log.d(TAG, "Processed CIRCLE gesture at center (${gesture.startX}, ${gesture.startY}) with radius ${gesture.radius}")
                            }
                        }
                    }
                }
            }
        }
    }

    private fun setupDefaultKeyMappings() {
        keyMappings.clear()
        keyMappings[KeyEvent.KEYCODE_W] = Pair(500f, 300f)
        keyMappings[KeyEvent.KEYCODE_A] = Pair(200f, 500f)
        keyMappings[KeyEvent.KEYCODE_S] = Pair(500f, 700f)
        keyMappings[KeyEvent.KEYCODE_D] = Pair(800f, 500f)
        keyMappings[KeyEvent.KEYCODE_SPACE] = Pair(900f, 800f)
        keyMappings[KeyEvent.KEYCODE_E] = Pair(1000f, 300f)
        keyMappings[KeyEvent.KEYCODE_R] = Pair(1100f, 400f)
        
        // Добавляем маппинги для геймпада
        keyMappings[KeyEvent.KEYCODE_BUTTON_A] = Pair(900f, 700f)
        keyMappings[KeyEvent.KEYCODE_BUTTON_B] = Pair(1000f, 600f)
        keyMappings[KeyEvent.KEYCODE_BUTTON_X] = Pair(800f, 600f)
        keyMappings[KeyEvent.KEYCODE_BUTTON_Y] = Pair(900f, 500f)
        keyMappings[KeyEvent.KEYCODE_BUTTON_L1] = Pair(200f, 200f)
        keyMappings[KeyEvent.KEYCODE_BUTTON_R1] = Pair(1000f, 200f)
        
        Log.i(TAG, "Default key mappings initialized")
    }

    private fun loadProfile(profileId: String) {
        val profile = ProfileManager.getProfile(profileId) ?: return
        currentProfile = profile
        
        // Загружаем маппинги из профиля
        keyMappings.clear()
        keyMappings.putAll(profile.keyMappings)
        
        // Пересоздаем оверлеи
        createOverlayButtons()
        
        Log.i(TAG, "Loaded profile: ${profile.name}")
        Toast.makeText(this, getString(R.string.profile_loaded, profile.name), Toast.LENGTH_SHORT).show()
    }

    private fun loadOverlayPositions() {
        val prefs = getSharedPreferences("GameMapperPrefs", Context.MODE_PRIVATE)
        val keys = prefs.all.keys.filter { it.startsWith("x_") }
        
        for (key in keys) {
            val keyCode = key.substring(2).toIntOrNull() ?: continue
            val x = prefs.getFloat(key, 0f)
            val y = prefs.getFloat("y_$keyCode", 0f)
            keyMappings[keyCode] = Pair(x, y)
        }
    }

    private fun saveOverlayPositions() {
        val prefs = getSharedPreferences("GameMapperPrefs", Context.MODE_PRIVATE)
        with(prefs.edit()) {
            for ((keyCode, position) in keyMappings) {
                putFloat("x_$keyCode", position.first)
                putFloat("y_$keyCode", position.second)
            }
            apply()
        }
        
        // Также сохраняем в текущий профиль, если он есть
        currentProfile?.let {
            it.keyMappings.clear()
            it.keyMappings.putAll(keyMappings)
            ProfileManager.saveProfile(it)
        }
    }

    private fun createOverlayButtons() {
        try {
            // Удаляем существующие оверлеи
            overlayButtons.values.forEach {
                 try {
                    windowManager.removeView(it)
                } catch (e: Exception) {
                    Log.e(TAG, "Error removing view: ${e.message}")
                }
            }
            overlayButtons.clear()
            
            mouseLookArea?.let {
                 try {
                    windowManager.removeView(it)
                } catch (e: Exception) {
                    Log.e(TAG, "Error removing mouse look area: ${e.message}")
                }
            }
            mouseLookArea = null
            
            // Создаем новые оверлеи
            for ((keyCode, position) in keyMappings) {
                createOverlayButton(keyCode, position.first, position.second)
            }
            
            createMouseLookArea(600f, 400f, 400f, 300f)
            Log.i(TAG, "Overlay buttons created successfully")
        } catch (e: Exception) {
            Log.e(TAG, "Failed to create overlay buttons: ${e.message}", e)
            Toast.makeText(this, R.string.overlay_creation_error, Toast.LENGTH_LONG).show()
        }
    }

    private fun createOverlayButton(keyCode: Int, x: Float, y: Float) {
        try {
            val displayMetrics = resources.displayMetrics
            val maxX = displayMetrics.widthPixels - 120
            val maxY = displayMetrics.heightPixels - 120
            val safeX = x.coerceIn(0f, maxX.toFloat())
            val safeY = y.coerceIn(0f, maxY.toFloat())
            
            // Получаем настройки кнопки из текущего профиля или используем значения по умолчанию
            val buttonSettings = currentProfile?.buttonSettings?.get(keyCode) ?: ButtonSettings()
            
            val button = CustomOverlayButton(this).apply {
                buttonText = KeyEvent.keyCodeToString(keyCode).replace("KEYCODE_", "")
                buttonColor = buttonSettings.color
                buttonSize = buttonSettings.size
                buttonShape = buttonSettings.shape
                buttonAlpha = buttonSettings.alpha
                buttonBorderColor = buttonSettings.borderColor
                buttonBorderWidth = buttonSettings.borderWidth
            }
            
            val params = WindowManager.LayoutParams(
                buttonSettings.size.toInt(), buttonSettings.size.toInt(),
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O)
                    WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY
                else
                    WindowManager.LayoutParams.TYPE_SYSTEM_ALERT,
                WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or 
                WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS,
                PixelFormat.TRANSLUCENT
            ).apply {
                gravity = Gravity.TOP or Gravity.START
                this.x = safeX.toInt()
                this.y = safeY.toInt()
            }
            
            button.setOnTouchListener(object : View.OnTouchListener {
                private var initialX = 0
                private var initialY = 0
                private var initialTouchX = 0f
                private var initialTouchY = 0f
                private var isMoving = false
                private var startClickTime = 0L
                private var isLongPress = false
                
                override fun onTouch(v: View, event: MotionEvent): Boolean {
                    when (event.action) {
                        MotionEvent.ACTION_DOWN -> {
                            initialX = params.x
                            initialY = params.y
                            initialTouchX = event.rawX
                            initialTouchY = event.rawY
                            startClickTime = System.currentTimeMillis()
                            isMoving = false
                            isLongPress = false
                            
                            // Запускаем таймер для определения долгого нажатия
                            scope.launch {
                                delay(500) // 500 мс для долгого нажатия
                                if (v.isPressed && !isMoving) {
                                    isLongPress = true
                                    // Показываем диалог настройки кнопки при долгом нажатии
                                    (context as? AppCompatActivity)?.runOnUiThread {
                                        showButtonSettingsDialog(button, keyCode)
                                    }
                                }
                            }
                            
                            return true
                        }
                        MotionEvent.ACTION_MOVE -> {
                            val deltaX = event.rawX - initialTouchX
                            val deltaY = event.rawY - initialTouchY
                            
                            // Определяем, движется ли кнопка
                            if (Math.abs(deltaX) > 10 || Math.abs(deltaY) > 10) {
                                isMoving = true
                            }
                            
                            if (isMoving) {
                                params.x = (initialX + deltaX).toInt().coerceIn(0, maxX)
                                params.y = (initialY + deltaY).toInt().coerceIn(0, maxY)
                                
                                scope.launch {
                                    try {
                                        windowManager.updateViewLayout(v, params)
                                        keyMappings[keyCode] = Pair(params.x.toFloat(), params.y.toFloat())
                                        saveOverlayPositions()
                                    } catch (e: Exception) {
                                        Log.e(TAG, "Failed to update button position: ${e.message}", e)
                                    }
                                }
                            }
                            return true
                        }
                        MotionEvent.ACTION_UP -> {
                            // Если это был клик (не перемещение и не долгое нажатие), то симулируем нажатие
                            if (!isMoving && !isLongPress && System.currentTimeMillis() - startClickTime < 200) {
                                scope.launch {
                                    performVirtualTouch(keyMappings[keyCode]!!.first, keyMappings[keyCode]!!.second)
                                    highlightButton(keyCode)
                                    
                                    // Также проверяем жесты
                                    processGesture(keyCode)
                                }
                            }
                            return true
                        }
                    }
                    return false
                }
            })
            
            windowManager.addView(button, params)
            overlayButtons[keyCode] = button
            Log.d(TAG, "Overlay button added for key: $keyCode")
        } catch (e: Exception) {
            Log.e(TAG, "Failed to create overlay button for key $keyCode: ${e.message}", e)
        }
    }

    private fun createMouseLookArea(x: Float, y: Float, width: Float, height: Float) {
        try {
            val view = View(this).apply {
                setBackgroundColor(Color.parseColor("#20000000"))
            }
            
            val params = WindowManager.LayoutParams(
                width.toInt(), height.toInt(),
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O)
                    WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY
                else
                    WindowManager.LayoutParams.TYPE_SYSTEM_ALERT,
                WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or
                WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS,
                PixelFormat.TRANSLUCENT
            ).apply {
                gravity = Gravity.TOP or Gravity.START
                this.x = x.toInt()
                this.y = y.toInt()
            }
            
            windowManager.addView(view, params)
            mouseLookArea = view
            Log.d(TAG, "Mouse look area created at ($x, $y) with size (${width}x$height)")
        } catch (e: Exception) {
            Log.e(TAG, "Failed to create mouse look area: ${e.message}", e)
        }
    }

    fun performVirtualTouch(x: Float, y: Float) {
        try {
            val method = injectInputEventMethod ?: throw IllegalStateException("Input method not initialized")
            val downTime = System.currentTimeMillis()
            
            val down = MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_DOWN, x, y, 0)
            val up = MotionEvent.obtain(downTime, downTime + 50, MotionEvent.ACTION_UP, x, y, 0)
            
            method.invoke(inputManager, down, 0)
            method.invoke(inputManager, up, 0)
            
            down.recycle()
            up.recycle()
            
            // Визуальный отклик
            scope.launch {
                try {
                    val touchIndicator = View(this@MappingService).apply {
                        setBackgroundResource(R.drawable.touch_indicator)
                    }
                    
                    val params = WindowManager.LayoutParams(
                        60, 60,
                        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O)
                            WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY
                        else
                            WindowManager.LayoutParams.TYPE_SYSTEM_ALERT,
                        WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or
                        WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE or
                        WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS,
                        PixelFormat.TRANSLUCENT
                    ).apply {
                        gravity = Gravity.TOP or Gravity.START
                        this.x = (x - 30).toInt()
                        this.y = (y - 30).toInt()
                    }
                    
                    windowManager.addView(touchIndicator, params)
                    delay(200)
                    windowManager.removeView(touchIndicator)
                } catch (e: Exception) {
                    Log.e(TAG, "Error showing touch indicator: ${e.message}", e)
                }
            }
            
            Log.d(TAG, "Virtual touch performed at ($x, $y)")
        } catch (e: Exception) {
            Log.e(TAG, "Failed to perform virtual touch: ${e.message}", e)
            Toast.makeText(this, R.string.touch_simulation_error, Toast.LENGTH_SHORT).show()
        }
    }

    fun simulateDrag(startX: Float, startY: Float, endX: Float, endY: Float) {
        try {
            val method = injectInputEventMethod ?: throw IllegalStateException("Input method not initialized")
            val downTime = System.currentTimeMillis()
            
            val down = MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_DOWN, startX, startY, 0)
            method.invoke(inputManager, down, 0)
            
            // Промежуточные точки для плавности
            val steps = 10
            val deltaX = (endX - startX) / steps
            val deltaY = (endY - startY) / steps
            
            for (i in 1 until steps) {
                val moveTime = downTime + i * 10
                val moveX = startX + deltaX * i
                val moveY = startY + deltaY * i
                val move = MotionEvent.obtain(downTime, moveTime, MotionEvent.ACTION_MOVE, moveX, moveY, 0)
                method.invoke(inputManager, move, 0)
                move.recycle()
                Thread.sleep(10)
            }
            
            val up = MotionEvent.obtain(downTime, downTime + 100, MotionEvent.ACTION_UP, endX, endY, 0)
            method.invoke(inputManager, up, 0)
            
            down.recycle()
            up.recycle()
            
            Log.d(TAG, "Drag performed from ($startX, $startY) to ($endX, $endY)")
        } catch (e: Exception) {
            Log.e(TAG, "Failed to simulate drag: ${e.message}", e)
            Toast.makeText(this, R.string.drag_simulation_error, Toast.LENGTH_SHORT).show()
        }
    }

    // Метод для выполнения долгого нажатия
    fun performLongPress(x: Float, y: Float, duration: Long) {
        try {
            val method = injectInputEventMethod ?: throw IllegalStateException("Input method not initialized")
            val downTime = System.currentTimeMillis()
            
            val down = MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_DOWN, x, y, 0)
            method.invoke(inputManager, down, 0)
            
            // Визуальный отклик
            scope.launch {
                try {
                    val touchIndicator = View(this@MappingService).apply {
                        setBackgroundResource(R.drawable.touch_indicator_long_press)
                    }
                    
                    val params = WindowManager.LayoutParams(
                        80, 80,
                        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) 
                            WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY
                        else 
                            WindowManager.LayoutParams.TYPE_SYSTEM_ALERT,
                        WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or 
                        WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE or
                        WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS,
                        PixelFormat.TRANSLUCENT
                    ).apply {
                        gravity = Gravity.TOP or Gravity.START
                        this.x = (x - 40).toInt()
                        this.y = (y - 40).toInt()
                    }
                    
                    windowManager.addView(touchIndicator, params)
                    delay(duration)
                    windowManager.removeView(touchIndicator)
                } catch (e: Exception) {
                    Log.e(TAG, "Error showing long press indicator: ${e.message}", e)
                }
            }
            
            // Ждем указанное время
            Thread.sleep(duration)
            
            // Отпускаем
            val up = MotionEvent.obtain(downTime, downTime + duration, MotionEvent.ACTION_UP, x, y, 0)
            method.invoke(inputManager, up, 0)
            
            down.recycle()
            up.recycle()
            
            Log.d(TAG, "Long press performed at ($x, $y) for $duration ms")
        } catch (e: Exception) {
            Log.e(TAG, "Failed to perform long press: ${e.message}", e)
            Toast.makeText(this, R.string.touch_simulation_error, Toast.LENGTH_SHORT).show()
        }
    }

    // Метод для выполнения мультитапа
    fun performMultiTap(x: Float, y: Float, count: Int, interval: Long = 100) {
        try {
            val method = injectInputEventMethod ?: throw IllegalStateException("Input method not initialized")
            
            for (i in 0 until count) {
                val downTime = System.currentTimeMillis()
                
                val down = MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_DOWN, x, y, 0)
                val up = MotionEvent.obtain(downTime, downTime + 50, MotionEvent.ACTION_UP, x, y, 0)
                
                method.invoke(inputManager, down, 0)
                method.invoke(inputManager, up, 0)
                
                down.recycle()
                up.recycle()
                
                if (i < count - 1) {
                    Thread.sleep(interval)
                }
            }
            
            Log.d(TAG, "Multi-tap performed at ($x, $y), count: $count")
        } catch (e: Exception) {
            Log.e(TAG, "Failed to perform multi-tap: ${e.message}", e)
            Toast.makeText(this, R.string.touch_simulation_error, Toast.LENGTH_SHORT).show()
        }
    }

    // Метод для выполнения круговых движений
    fun performCircularMotion(centerX: Float, centerY: Float, radius: Float, duration: Long = 1000) {
        try {
            val method = injectInputEventMethod ?: throw IllegalStateException("Input method not initialized")
            val downTime = System.currentTimeMillis()
            val steps = 36 // количество шагов для полного круга
            val angleStep = 2 * Math.PI / steps
            val timeStep = duration / steps
            
            // Начальная точка
            var currentX = centerX + radius
            var currentY = centerY
            
            // Начальное касание
            val down = MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_DOWN, currentX, currentY, 0)
            method.invoke(inputManager, down, 0)
            down.recycle()
            
            // Движение по кругу
            for (i in 1..steps) {
                val angle = i * angleStep
                currentX = centerX + (radius * Math.cos(angle)).toFloat()
                currentY = centerY + (radius * Math.sin(angle)).toFloat()
                
                val move = MotionEvent.obtain(
                    downTime, 
                    downTime + (i * timeStep), 
                    MotionEvent.ACTION_MOVE, 
                    currentX, 
                    currentY, 
                    0
                )
                method.invoke(inputManager, move, 0)
                move.recycle()
                
                Thread.sleep(timeStep)
            }
            
            // Завершающее отпускание
            val up = MotionEvent.obtain(
                downTime, 
                downTime + duration, 
                MotionEvent.ACTION_UP, 
                currentX, 
                currentY, 
                0
            )
            method.invoke(inputManager, up, 0)
            up.recycle()
            
            Log.d(TAG, "Circular motion performed at center ($centerX, $centerY) with radius $radius")
        } catch (e: Exception) {
            Log.e(TAG, "Failed to perform circular motion: ${e.message}", e)
            Toast.makeText(this, R.string.touch_simulation_error, Toast.LENGTH_SHORT).show()
        }
    }

    // Добавим метод для показа диалога настройки кнопки
    private fun showButtonSettingsDialog(button: CustomOverlayButton, keyCode: Int) {
        val activity = context as? AppCompatActivity ?: return
        
        ButtonSettingsDialog(activity, button) { updatedButton ->
            // Сохраняем настройки кнопки в профиль
            currentProfile?.let { profile ->
                val settings = ButtonSettings(
                    shape = updatedButton.buttonShape,
                    color = updatedButton.buttonColor,
                    size = updatedButton.buttonSize,
                    alpha = updatedButton.buttonAlpha,
                    borderColor = updatedButton.buttonBorderColor,
                    borderWidth = updatedButton.buttonBorderWidth
                )
                
                profile.buttonSettings[keyCode] = settings
                ProfileManager.saveProfile(profile)
                
                // Обновляем размер в WindowManager, если он изменился
                val oldButton = overlayButtons[keyCode]
                if (oldButton != null && oldButton.buttonSize != updatedButton.buttonSize) {
                    val params = WindowManager.LayoutParams(
                        updatedButton.buttonSize.toInt(), updatedButton.buttonSize.toInt(),
                        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O)
                            WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY
                        else
                            WindowManager.LayoutParams.TYPE_SYSTEM_ALERT,
                        WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or 
                        WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS,
                        PixelFormat.TRANSLUCENT
                    ).apply {
                        gravity = Gravity.TOP or Gravity.START
                        val position = keyMappings[keyCode] ?: Pair(0f, 0f)
                        this.x = position.first.toInt()
                        this.y = position.second.toInt()
                    }
                    
                    try {
                        windowManager.updateViewLayout(oldButton, params)
                    } catch (e: Exception) {
                        Log.e(TAG, "Failed to update button layout: ${e.message}", e)
                    }
                }
            }
        }.show()
    }

    override fun onDestroy() {
        super.onDestroy()
        
        // Очищаем все ресурсы
        try {
            // Удаляем все оверлеи
            overlayButtons.values.forEach {
                try {
                    windowManager.removeView(it)
                } catch (e: Exception) {
                    Log.e(TAG, "Error removing view: ${e.message}")
                }
            }
            overlayButtons.clear()
            
            mouseLookArea?.let {
                try {
                    windowManager.removeView(it)
                } catch (e: Exception) {
                    Log.e(TAG, "Error removing mouse look area: ${e.message}")
                }
            }
            mouseLookArea = null
            
            // Отменяем все корутины
            scope.cancel()
            
            isServiceActive = false
            Log.i(TAG, "Service destroyed")
        } catch (e: Exception) {
            Log.e(TAG, "Error during service destruction: ${e.message}", e)
        }
    }
}

```

## kotlin File: `MainActivity.kt`
```kotlin
package com.example.gamemapper

import android.app.AlertDialog
import android.content.Context
import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.provider.Settings
import android.view.View
import android.widget.Button
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    private lateinit var startServiceButton: Button
    private lateinit var stopServiceButton: Button
    private lateinit var createMappingButton: Button
    private lateinit var editMappingButton: Button
    private lateinit var loadMappingButton: Button
    private lateinit var saveMappingButton: Button
    private lateinit var playButton: Button
    private lateinit var gamepadConfigButton: Button
    private lateinit var profilesButton: Button
    private lateinit var statusTextView: TextView
    private lateinit var recordIndicator: View
    
    private val openDocumentLauncher = registerForActivityResult(
        ActivityResultContracts.OpenDocument()
    ) { uri ->
        uri?.let {
            contentResolver.takePersistableUriPermission(
                it,
                Intent.FLAG_GRANT_READ_URI_PERMISSION
            )
            if (RecorderManager.loadRecording(this, it)) {
                Toast.makeText(this, R.string.mapping_loaded, Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    private val createDocumentLauncher = registerForActivityResult(
        ActivityResultContracts.CreateDocument("text/plain")
    ) { uri ->
        uri?.let {
            contentResolver.takePersistableUriPermission(
                it,
                Intent.FLAG_GRANT_WRITE_URI_PERMISSION
            )
            if (RecorderManager.saveRecording(this, it)) {
                Toast.makeText(this, R.string.mapping_saved, Toast.LENGTH_SHORT).show()
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        startServiceButton = findViewById(R.id.startServiceButton)
        stopServiceButton = findViewById(R.id.stopServiceButton)
        createMappingButton = findViewById(R.id.createMappingButton)
        editMappingButton = findViewById(R.id.editMappingButton)
        loadMappingButton = findViewById(R.id.loadMappingButton)
        saveMappingButton = findViewById(R.id.saveMappingButton)
        playButton = findViewById(R.id.playButton)
        gamepadConfigButton = findViewById(R.id.gamepadConfigButton)
        profilesButton = findViewById(R.id.profilesButton)
        statusTextView = findViewById(R.id.statusTextView)
        recordIndicator = findViewById(R.id.recordIndicator)
        
        startServiceButton.setOnClickListener {
            if (!isAccessibilityServiceEnabled()) {
                promptEnableService()
            } else {
                startService(Intent(this, MappingService::class.java))
                Toast.makeText(this, R.string.service_started, Toast.LENGTH_SHORT).show()
                updateUI()
            }
        }
        
        stopServiceButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                stopService(Intent(this, MappingService::class.java).apply { action = "STOP_SERVICE" })
                updateUI()
            }
        }
        
        createMappingButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                RecorderManager.startRecording()
                recordIndicator.visibility = View.VISIBLE
                Toast.makeText(this, R.string.creating_mapping, Toast.LENGTH_SHORT).show()
            } else {
                promptEnableService()
            }
        }
        
        editMappingButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                if (RecorderManager.isRecording()) {
                    RecorderManager.stopRecording()
                    recordIndicator.visibility = View.GONE
                }
                startActivity(Intent(this, EditMappingActivity::class.java))
            } else {
                promptEnableService()
            }
        }
        
        loadMappingButton.setOnClickListener {
            openDocumentLauncher.launch(arrayOf("text/plain"))
        }
        
        saveMappingButton.setOnClickListener {
            if (RecorderManager.getRecordedActions().isEmpty()) {
                Toast.makeText(this, R.string.no_actions_to_save, Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            createDocumentLauncher.launch("gamemapper_mapping_${System.currentTimeMillis()}.gmp")
        }
        
        playButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                val service = getSystemService(Context.ACCESSIBILITY_SERVICE) as? MappingService
                service?.let {
                     if (RecorderManager.getRecordedActions().isEmpty()) {
                        Toast.makeText(this, R.string.no_actions_to_play, Toast.LENGTH_SHORT).show()
                    } else {
                        RecorderManager.playRecording(it, this)
                     }
                }
            } else {
                promptEnableService()
            }
        }
        
        gamepadConfigButton.setOnClickListener {
            startActivity(Intent(this, GamepadConfigActivity::class.java))
        }
        
        profilesButton.setOnClickListener {
            startActivity(Intent(this, ProfileManagementActivity::class.java))
        }
        
        // Добавляем кнопку для настройки жестов
        val gestureConfigButton = findViewById<Button>(R.id.gestureConfigButton)
        gestureConfigButton.setOnClickListener {
            // Получаем текущий активный профиль
            val profiles = ProfileManager.getAllProfiles()
            if (profiles.isEmpty()) {
                Toast.makeText(this, R.string.no_profiles_available, Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            
            // Если есть только один профиль, открываем настройку жестов для него
            if (profiles.size == 1) {
                val intent = Intent(this, GestureConfigActivity::class.java).apply {
                    putExtra("profile_id", profiles[0].id)
                }
                startActivity(intent)
                return@setOnClickListener
            }
            
            // Если есть несколько профилей, показываем диалог выбора
            val profileNames = profiles.map { it.name }.toTypedArray()
            AlertDialog.Builder(this)
                .setTitle(R.string.select_profile_for_gestures)
                .setItems(profileNames) { _, which ->
                    val selectedProfile = profiles[which]
                    val intent = Intent(this, GestureConfigActivity::class.java).apply {
                        putExtra("profile_id", selectedProfile.id)
                    }
                    startActivity(intent)
                }
                .setNegativeButton(R.string.cancel, null)
                .show()
        }
        
        // Инициализируем PersistenceManager
        PersistenceManager.init(applicationContext)
        
        // Загружаем профили
        ProfileManager.loadProfiles()
        
        // Проверяем разрешения при первом запуске
        if (savedInstanceState == null) {
            checkPermissions()
        }
    }

    override fun onResume() {
        super.onResume()
        updateUI()
    }
    
    private fun updateUI() {
        val serviceEnabled = isAccessibilityServiceEnabled()
        stopServiceButton.isEnabled = serviceEnabled
        createMappingButton.isEnabled = serviceEnabled
        editMappingButton.isEnabled = serviceEnabled
        loadMappingButton.isEnabled = true
        saveMappingButton.isEnabled = RecorderManager.getRecordedActions().isNotEmpty()
        playButton.isEnabled = serviceEnabled && RecorderManager.getRecordedActions().isNotEmpty()
        gamepadConfigButton.isEnabled = serviceEnabled
        profilesButton.isEnabled = true
        findViewById<Button>(R.id.gestureConfigButton).isEnabled = ProfileManager.getAllProfiles().isNotEmpty()
        
        statusTextView.text = getString(
            if (serviceEnabled) R.string.service_running else R.string.service_not_running
        ).let { "${getString(R.string.status_label)} $it" }
        
        recordIndicator.visibility = if (RecorderManager.isRecording()) View.VISIBLE else View.GONE
    }
    
    private fun isAccessibilityServiceEnabled(): Boolean {
        val service = "$packageName/.MappingService"
        val enabledServices = Settings.Secure.getString(
            contentResolver,
            Settings.Secure.ENABLED_ACCESSIBILITY_SERVICES
        )
        return enabledServices?.contains(service) == true
    }
    
    private fun promptEnableService() {
        AlertDialog.Builder(this)
            .setTitle(R.string.service_required)
            .setMessage(R.string.service_explanation)
            .setPositiveButton(R.string.go_to_settings) { _, _ ->
                startActivity(Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS))
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun checkPermissions() {
        if (!Settings.canDrawOverlays(this)) {
            AlertDialog.Builder(this)
                .setTitle(R.string.overlay_permission_required)
                .setMessage(R.string.overlay_permission_explanation)
                .setPositiveButton(R.string.go_to_settings) { _, _ ->
                    val intent = Intent(
                        Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                        Uri.parse("package:$packageName")
                    )
                    startActivity(intent)
                }
                .setNegativeButton(R.string.cancel, null)
                .show()
        }
    }
}

```

## xml File: `AndroidManifest.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.gamemapper">

    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.GameMapper">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity
            android:name=".GamepadConfigActivity"
            android:exported="false" />

        <activity
            android:name=".ProfileManagementActivity"
            android:exported="false" />

        <activity
            android:name=".EditMappingActivity"
            android:exported="false" />

        <activity
            android:name=".GestureConfigActivity"
            android:exported="false" />

        <service
            android:name=".MappingService"
            android:exported="false"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>
            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/accessibility_service_config" />
        </service>
    </application>

</manifest>

```

## xml File: `accessibility_service_config.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:description="@string/app_name"
    android:accessibilityEventTypes="typeWindowStateChanged|typeViewClicked|typeViewFocused"
    android:accessibilityFlags="flagRequestFilterKeyEvents|flagRequestTouchExplorationMode"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:notificationTimeout="100"
    android:canRequestFilterKeyEvents="true"
    android:canPerformGestures="true" />

```

## xml File: `strings.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">GameMapperPro</string>
    <string name="status_label">Статус:</string>
    <string name="service_running">Сервис запущен</string>
    <string name="service_not_running">Сервис не запущен</string>
    <string name="start_service">Запустить сервис</string>
    <string name="stop_service">Остановить сервис</string>
    <string name="create_mapping">Создать маппинг</string>
    <string name="edit_mapping">Редактировать</string>
    <string name="load_mapping">Загрузить</string>
    <string name="save_mapping">Сохранить</string>
    <string name="play_recording">Воспроизвести</string>
    <string name="config_gamepad">Настройка геймпада</string>
    <string name="manage_profiles">Управление профилями</string>
    <string name="service_started">Сервис запущен</string>
    <string name="service_already_running">Сервис уже запущен</string>
    <string name="creating_mapping">Создание маппинга…</string>
    <string name="editing_mapping">Редактирование маппинга…</string>
    <string name="enable_service_prompt">Включите сервис в настройках доступности</string>
    <string name="service_required">Требуется сервис доступности</string>
    <string name="service_explanation">Для работы приложения необходимо включить сервис доступности. Это позволит перехватывать ввод и симулировать касания.</string>
    <string name="go_to_settings">Перейти в настройки</string>
    <string name="cancel">Отмена</string>
    <string name="overlay_permission_required">Требуется разрешение наложения</string>
    <string name="overlay_permission_explanation">Для отображения кнопок поверх других приложений необходимо разрешение на наложение.</string>
    <string name="no_actions_to_play">Нет действий для воспроизведения</string>
    <string name="no_actions_to_save">Нет действий для сохранения</string>
    <string name="mapping_loaded">Маппинг загружен</string>
    <string name="mapping_saved">Маппинг сохранен</string>
    <string name="service_start_error">Ошибка запуска сервиса</string>
    <string name="overlay_creation_error">Ошибка создания наложений</string>
    <string name="touch_simulation_error">Ошибка симуляции касания</string>
    <string name="drag_simulation_error">Ошибка симуляции перетаскивания</string>
    <string name="playback_error">Ошибка воспроизведения</string>
    <string name="playback_action_error">Ошибка воспроизведения действия</string>
    <string name="save_recording_error">Ошибка сохранения записи</string>
    <string name="load_recording_error">Ошибка загрузки записи</string>
    <string name="notification_title">GameMapperPro активен</string>
    <string name="notification_text">Сервис маппинга запущен</string>
    <string name="notification_channel_name">Сервис маппинга</string>
    <string name="notification_channel_description">Уведомление о работе сервиса маппинга</string>
    <string name="stop">Остановить</string>
    <string name="gamepad_configuration">Настройка геймпада</string>
    <string name="assign_button">Назначить кнопку</string>
    <string name="press_gamepad_button">Нажмите кнопку на геймпаде</string>
    <string name="button_assigned">Кнопка %s назначена</string>
    <string name="invalid_position">Неверная позиция</string>
    <string name="set_position">Установить позицию</string>
    <string name="ok">OK</string>
    <string name="profile_management">Управление профилями</string>
    <string name="add_profile">Добавить профиль</string>
    <string name="profile_name">Имя профиля</string>
    <string name="profile_name_hint">Введите имя профиля</string>
    <string name="create_profile">Создать профиль</string>
    <string name="create">Создать</string>
    <string name="edit_profile">Редактировать профиль</string>
    <string name="save">Сохранить</string>
    <string name="delete_profile">Удалить профиль</string>
    <string name="delete_profile_confirmation">Вы уверены, что хотите удалить профиль \"%s\"?</string>
    <string name="delete">Удалить</string>
    <string name="edit">Редактировать</string>
    <string name="select_app">Выбрать приложение</string>
    <string name="assign_app">Назначить приложение</string>
    <string name="app_assigned_to_profile">%1$s назначено профилю %2$s</string>
    <string name="profile_created">Профиль \"%s\" создан</string>
    <string name="profile_updated">Профиль \"%s\" обновлен</string>
    <string name="profile_deleted">Профиль \"%s\" удален</string>
    <string name="profile_name_required">Имя профиля обязательно</string>
    <string name="profile_loaded">Профиль \"%s\" загружен</string>
    <string name="profile_activated">Профиль \"%s\" активирован</string>
    <string name="x_position">Позиция X:</string>
    <string name="y_position">Позиция Y:</string>
    <string name="button_shape">Форма кнопки:</string>
    <string name="button_color">Цвет кнопки:</string>
    <string name="button_size">Размер кнопки:</string>
    <string name="circle">Круг</string>
    <string name="square">Квадрат</string>
    <string name="rounded">Скругленный</string>
    <string name="button_transparency">Прозрачность:</string>
    <string name="button_settings">Настройки кнопки</string>
    <string name="configure_gestures">Настройка жестов</string>
    <string name="gesture_configuration">Настройка жестов</string>
    <string name="add_gesture">Добавить жест</string>
    <string name="edit_gesture">Редактировать жест</string>
    <string name="gesture_type">Тип жеста</string>
    <string name="key_code">Код клавиши</string>
    <string name="start_coordinates">Начальные координаты</string>
    <string name="end_coordinates">Конечные координаты</string>
    <string name="duration_ms">Длительность (мс)</string>
    <string name="tap_count">Количество нажатий</string>
    <string name="radius">Радиус</string>
    <string name="gesture_added">Жест добавлен</string>
    <string name="gesture_updated">Жест обновлен</string>
    <string name="gesture_config_error">Ошибка настройки жеста</string>
    <string name="select_profile_for_gestures">Выберите профиль для настройки жестов</string>
    <string name="no_profiles_available">Нет доступных профилей</string>
    <string name="duration">Длительность</string>
    <string name="gesture_config_for">Настройка жестов для %s</string>
    <string name="select">Выбрать</string>
    <string name="playback_completed">Воспроизведение завершено</string>
    <string name="no_app_assigned">Приложение не назначено</string>
</resources>

```

## xml File: `activity_edit_mapping.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/keyList"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@+id/addButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginBottom="16dp"/>

    <Button
        android:id="@+id/addButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Add Mapping"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

## xml File: `activity_gamepad_config.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <TextView
        android:id="@+id/titleTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/gamepad_configuration"
        android:textSize="20sp"
        android:textStyle="bold"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/gamepadListView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="16dp"
        app:layout_constraintTop_toBottomOf="@+id/titleTextView"
        app:layout_constraintBottom_toTopOf="@+id/assignButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/assignButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/assign_button"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>

```

## xml File: `activity_gesture_config.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <TextView
        android:id="@+id/titleTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/gesture_configuration"
        android:textSize="20sp"
        android:textStyle="bold"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/gesturesRecyclerView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="16dp"
        app:layout_constraintTop_toBottomOf="@+id/titleTextView"
        app:layout_constraintBottom_toTopOf="@+id/addGestureButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/addGestureButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/add_gesture"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>

```

## xml File: `activity_main.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <TextView
        android:id="@+id/titleTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/app_name"
        android:textSize="24sp"
        android:textStyle="bold"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <TextView
        android:id="@+id/statusTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/status_label"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@+id/titleTextView"
        app:layout_constraintStart_toStartOf="parent" />

    <View
        android:id="@+id/recordIndicator"
        android:layout_width="16dp"
        android:layout_height="16dp"
        android:background="@drawable/record_indicator"
        android:layout_marginStart="8dp"
        android:visibility="gone"
        app:layout_constraintTop_toTopOf="@+id/statusTextView"
        app:layout_constraintBottom_toBottomOf="@+id/statusTextView"
        app:layout_constraintStart_toEndOf="@+id/statusTextView" />

    <Button
        android:id="@+id/startServiceButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="@string/start_service"
        android:layout_marginTop="24dp"
        app:layout_constraintTop_toBottomOf="@+id/statusTextView"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/stopServiceButton"
        app:layout_constraintHorizontal_chainStyle="spread" />

    <Button
        android:id="@+id/stopServiceButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="@string/stop_service"
        android:layout_marginStart="8dp"
        app:layout_constraintTop_toTopOf="@+id/startServiceButton"
        app:layout_constraintStart_toEndOf="@+id/startServiceButton"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/createMappingButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="@string/create_mapping"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@+id/startServiceButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/editMappingButton"
        app:layout_constraintHorizontal_chainStyle="spread" />

    <Button
        android:id="@+id/editMappingButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="@string/edit_mapping"
        android:layout_marginStart="8dp"
        app:layout_constraintTop_toTopOf="@+id/createMappingButton"
        app:layout_constraintStart_toEndOf="@+id/createMappingButton"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/loadMappingButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="@string/load_mapping"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@+id/createMappingButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/saveMappingButton"
        app:layout_constraintHorizontal_chainStyle="spread" />

    <Button
        android:id="@+id/saveMappingButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="@string/save_mapping"
        android:layout_marginStart="8dp"
        app:layout_constraintTop_toTopOf="@+id/loadMappingButton"
        app:layout_constraintStart_toEndOf="@+id/loadMappingButton"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/playButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/play_recording"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@+id/loadMappingButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/gamepadConfigButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/config_gamepad"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@+id/playButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/profilesButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/manage_profiles"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@+id/gamepadConfigButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/gestureConfigButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/configure_gestures"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@+id/profilesButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>

```

## xml File: `activity_profile_management.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <TextView
        android:id="@+id/titleTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/profile_management"
        android:textSize="20sp"
        android:textStyle="bold"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/profilesRecyclerView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="16dp"
        app:layout_constraintTop_toBottomOf="@+id/titleTextView"
        app:layout_constraintBottom_toTopOf="@+id/addProfileButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/addProfileButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/add_profile"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>

```

## xml File: `dialog_button_settings.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/button_shape"
            android:layout_marginBottom="8dp" />
            
        <RadioGroup
            android:id="@+id/shapeRadioGroup"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="16dp">
            
            <RadioButton
                android:id="@+id/shapeCircle"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/circle"
                android:checked="true" />
                
            <RadioButton
                android:id="@+id/shapeSquare"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/square"
                android:layout_marginStart="8dp" />
                
            <RadioButton
                android:id="@+id/shapeRounded"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/rounded"
                android:layout_marginStart="8dp" />
        </RadioGroup>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/button_color"
            android:layout_marginBottom="8dp" />
            
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="16dp">
            
            <View
                android:id="@+id/colorPreview"
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:background="#FFFFFF" />
                
            <Button
                android:id="@+id/selectColorButton"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/select"
                android:layout_marginStart="16dp" />
        </LinearLayout>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/button_size"
            android:layout_marginBottom="8dp" />
            
        <SeekBar
            android:id="@+id/sizeSeekBar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:max="200"
            android:progress="120"
            android:layout_marginBottom="16dp" />
            
        <TextView
            android:id="@+id/sizeValueText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="120"
            android:gravity="center"
            android:layout_marginBottom="16dp" />
            
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/button_transparency"
            android:layout_marginBottom="8dp" />
            
        <SeekBar
            android:id="@+id/transparencySeekBar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:max="255"
            android:progress="128"
            android:layout_marginBottom="16dp" />
            
        <TextView
            android:id="@+id/transparencyValueText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="50%"
            android:gravity="center"
            android:layout_marginBottom="16dp" />
    </LinearLayout>
</ScrollView>

```

## xml File: `dialog_create_profile.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/profile_name"
        android:layout_marginBottom="8dp" />

    <EditText
        android:id="@+id/profileNameEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="text"
        android:hint="@string/profile_name_hint" />

</LinearLayout>

```

## xml File: `item_profile.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp">

        <TextView
            android:id="@+id/profileNameTextView"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:textStyle="bold"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/editButton"
            android:layout_marginEnd="8dp" />

        <TextView
            android:id="@+id/appNameTextView"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textSize="14sp"
            android:textStyle="italic"
            android:layout_marginTop="4dp"
            app:layout_constraintTop_toBottomOf="@+id/profileNameTextView"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/editButton"
            android:layout_marginEnd="8dp" />

        <ImageButton
            android:id="@+id/editButton"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:src="@drawable/ic_edit"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:contentDescription="@string/edit"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/deleteButton"
            android:layout_marginEnd="8dp" />

        <ImageButton
            android:id="@+id/deleteButton"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:src="@drawable/ic_delete"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:contentDescription="@string/delete"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintEnd_toEndOf="parent" />

        <Button
            android:id="@+id/assignAppButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/assign_app"
            android:layout_marginTop="8dp"
            app:layout_constraintTop_toBottomOf="@+id/appNameTextView"
            app:layout_constraintStart_toStartOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</androidx.cardview.widget.CardView>

```

## xml File: `item_key_mapping.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp">

        <TextView
            android:id="@android:id/text1"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:textStyle="bold"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toStartOf="@android:id/button1"
            android:layout_marginEnd="8dp" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="X:"
            android:layout_marginTop="8dp"
            app:layout_constraintTop_toBottomOf="@android:id/text1"
            app:layout_constraintStart_toStartOf="parent" />

        <EditText
            android:id="@android:id/edit"
            android:layout_width="80dp"
            android:layout_height="wrap_content"
            android:inputType="numberDecimal"
            android:layout_marginStart="24dp"
            android:layout_marginTop="4dp"
            app:layout_constraintTop_toBottomOf="@android:id/text1"
            app:layout_constraintStart_toStartOf="parent" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Y:"
            android:layout_marginTop="8dp"
            android:layout_marginStart="16dp"
            app:layout_constraintTop_toBottomOf="@android:id/text1"
            app:layout_constraintStart_toEndOf="@android:id/edit" />

        <EditText
            android:id="@android:id/custom"
            android:layout_width="80dp"
            android:layout_height="wrap_content"
            android:inputType="numberDecimal"
            android:layout_marginStart="24dp"
            android:layout_marginTop="4dp"
            app:layout_constraintTop_toBottomOf="@android:id/text1"
            app:layout_constraintStart_toEndOf="@android:id/edit" />

        <Button
            android:id="@android:id/button1"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="@drawable/ic_delete"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintEnd_toEndOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</androidx.cardview.widget.CardView>

```

## xml File: `item_gesture.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp">

        <TextView
            android:id="@+id/gestureTypeText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:textStyle="bold"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/editButton"
            android:layout_marginEnd="8dp" />

        <TextView
            android:id="@+id/keyCodeText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textSize="16sp"
            android:textStyle="italic"
            android:layout_marginTop="4dp"
            app:layout_constraintTop_toBottomOf="@+id/gestureTypeText"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/editButton"
            android:layout_marginEnd="8dp" />

        <TextView
            android:id="@+id/gestureDetailsText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textSize="14sp"
            android:layout_marginTop="4dp"
            app:layout_constraintTop_toBottomOf="@+id/keyCodeText"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/editButton"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/editButton"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="@drawable/ic_edit"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/deleteButton"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/deleteButton"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="@drawable/ic_delete"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintEnd_toEndOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</androidx.cardview.widget.CardView>

```

## xml File: `item_gamepad_button.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp">

        <TextView
            android:id="@+id/buttonNameText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:textStyle="bold"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/deleteButton"
            android:layout_marginEnd="8dp" />

        <TextView
            android:id="@+id/positionText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textSize="14sp"
            android:layout_marginTop="4dp"
            app:layout_constraintTop_toBottomOf="@+id/buttonNameText"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/deleteButton"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/deleteButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/delete"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintEnd_toEndOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</androidx.cardview.widget.CardView>

```

## xml File: `dialog_position_input.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/x_position"
        android:layout_marginBottom="8dp" />

    <EditText
        android:id="@+id/xPositionInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="numberDecimal"
        android:layout_marginBottom="16dp" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/y_position"
        android:layout_marginBottom="8dp" />

    <EditText
        android:id="@+id/yPositionInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="numberDecimal" />
</LinearLayout>

```

## xml File: `dialog_gesture_config.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/gesture_type"
            android:layout_marginBottom="8dp" />
            
        <Spinner
            android:id="@+id/gestureTypeSpinner"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp" />
            
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/key_code"
            android:layout_marginBottom="8dp" />
            
        <Spinner
            android:id="@+id/keyCodeSpinner"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp" />
            
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/start_coordinates"
            android:layout_marginBottom="8dp" />
            
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="16dp">
            
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="X:"
                android:layout_marginEnd="8dp" />
                
            <EditText
                android:id="@+id/startXEdit"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:inputType="numberDecimal"
                android:layout_marginEnd="16dp" />
                
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Y:"
                android:layout_marginEnd="8dp" />
                
            <EditText
                android:id="@+id/startYEdit"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:inputType="numberDecimal" />
        </LinearLayout>
        
        <LinearLayout
            android:id="@+id/endCoordinatesLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:visibility="gone">
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/end_coordinates"
                android:layout_marginBottom="8dp" />
                
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:layout_marginBottom="16dp">
                
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="X:"
                    android:layout_marginEnd="8dp" />
                    
                <EditText
                    android:id="@+id/endXEdit"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:inputType="numberDecimal"
                    android:layout_marginEnd="16dp" />
                    
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Y:"
                    android:layout_marginEnd="8dp" />
                    
                <EditText
                    android:id="@+id/endYEdit"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:inputType="numberDecimal" />
            </LinearLayout>
        </LinearLayout>
        
        <LinearLayout
            android:id="@+id/durationLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:visibility="gone">
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/duration_ms"
                android:layout_marginBottom="8dp" />
                
            <EditText
                android:id="@+id/durationEdit"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="number"
                android:text="500"
                android:layout_marginBottom="16dp" />
        </LinearLayout>
        
        <LinearLayout
            android:id="@+id/tapCountLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:visibility="gone">
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/tap_count"
                android:layout_marginBottom="8dp" />
                
            <EditText
                android:id="@+id/tapCountEdit"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="number"
                android:text="2"
                android:layout_marginBottom="16dp" />
        </LinearLayout>
        
        <LinearLayout
            android:id="@+id/radiusLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:visibility="gone">
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/radius"
                android:layout_marginBottom="8dp" />
                
            <EditText
                android:id="@+id/radiusEdit"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="numberDecimal"
                android:text="100"
                android:layout_marginBottom="16dp" />
        </LinearLayout>
    </LinearLayout>
</ScrollView>

```

## xml File: `ic_delete.xml`
```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#000000"
        android:pathData="M6,19c0,1.1 0.9,2 2,2h8c1.1,0 2,-0.9 2,-2V7H6v12zM19,4h-3.5l-1,-1h-5l-1,1H5v2h14V4z"/>
</vector>

```

## xml File: `ic_edit.xml`
```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#000000"
        android:pathData="M3,17.25V21h3.75L17.81,9.94l-3.75,-3.75L3,17.25zM20.71,7.04c0.39,-0.39 0.39,-1.02 0,-1.41l-2.34,-2.34c-0.39,-0.39 -1.02,-0.39 -1.41,0l-1.83,1.83 3.75,3.75 1.83,-1.83z"/>
</vector>

```

## xml File: `ic_gamepad.xml`
```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M21,6L3,6c-1.1,0 -2,0.9 -2,2v8c0,1.1 0.9,2 2,2h18c1.1,0 2,-0.9 2,-2L23,8c0,-1.1 -0.9,-2 -2,-2zM11,13L8,13v3L6,16v-3L3,13v-2h3L6,8h2v3h3v2zM15.5,15c-0.83,0 -1.5,-0.67 -1.5,-1.5s0.67,-1.5 1.5,-1.5 1.5,0.67 1.5,1.5 -0.67,1.5 -1.5,1.5zM19.5,12c-0.83,0 -1.5,-0.67 -1.5,-1.5S18.67,9 19.5,9s1.5,0.67 1.5,1.5 -0.67,1.5 -1.5,1.5z"/>
</vector>

```

## xml File: `ic_stop.xml`
```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M6,6h12v12H6z"/>
</vector>

```

## xml File: `record_indicator.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="#FFFF0000" />
</shape>

```

## xml File: `touch_indicator_long_press.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="#800000FF" />
    <stroke android:width="2dp" android:color="#FFFFFF" />
    <size android:width="80dp" android:height="80dp" />
</shape>

```

## xml File: `touch_indicator.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="#8000FF00" />
    <size android:width="60dp" android:height="60dp" />
</shape>

```

