# firebase-sharing-keychain

```swift
//
//  mywidgets.swift
//  mywidgets
//
//  Created by paige shin on 2023/01/10.
//

import WidgetKit
import SwiftUI
import Firebase
import FirebaseAuth 

struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date())
    }

    func getSnapshot(in context: Context, completion: @escaping (SimpleEntry) -> ()) {
        let entry = SimpleEntry(date: Date())
        completion(entry)
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        let date = Date()
        let nextUpdate = Calendar.current.date(byAdding: .minute, value: 15, to: date)!
        fetchFromDB { work in
            let entry = SimpleEntry(date: date, workoutData: work)
            let timeline = Timeline(entries: [entry], policy: .after(nextUpdate))
            completion(timeline)
        }
    }
    
    func fetchFromDB(completion: @escaping(WorkoutModel) -> Void) {
        guard let _  = Auth.auth().currentUser else {
            completion(WorkoutModel(daysValue: [], maxValue: 0, error: "Please Login!"))
            return
        }
        let db = Firestore.firestore().collection("Workout").document("O8MjC6kqz12CIGdgFNqJ")
        db.getDocument { snap, err in
            guard let doc = snap?.data() else {
                completion(WorkoutModel(daysValue: [], maxValue: 0, error: err?.localizedDescription ?? ""))
                return
            }
            let maxValue = doc["Max"] as? String ?? ""
            var dayValue: [Int] = []
            for index in 1...7 {
                let day = doc["Day\(index)"] as? String ?? ""
                dayValue.append(Int(day) ?? 0)
            }
            completion(WorkoutModel(daysValue: dayValue, maxValue: Int(maxValue) ?? 0))
        }
    }
}

struct WorkoutModel: Identifiable {
    var id = UUID().uuidString
    var daysValue: [Int]
    var maxValue: Int
    var error: String = ""
}

struct SimpleEntry: TimelineEntry {
    let date: Date
    var workoutData: WorkoutModel?
}

struct mywidgetsEntryView : View {
    var entry: Provider.Entry

    var body: some View {
        ZStack {
            if let work = entry.workoutData {
                if work.error == "" {
                    Text("\(work.maxValue)")
                } else {
                    Text(work.error)
                        .padding()
                }
            } else {
                Text("Fetching Data...")
            }
        }
    }
}

struct mywidgets: Widget {
    
    init() {
        FirebaseApp.configure()
        do {
            try Auth.auth().useUserAccessGroup("teamid.com.paigesoftware.firebase-widgets")
        } catch {
            print(error.localizedDescription)
        }
    }
    
    let kind: String = "mywidgets"

    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            mywidgetsEntryView(entry: entry)
        }
        .supportedFamilies([.systemLarge])
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}

struct mywidgets_Previews: PreviewProvider {
    static var previews: some View {
        mywidgetsEntryView(entry: SimpleEntry(date: Date()))
            .previewContext(WidgetPreviewContext(family: .systemSmall))
    }
}

```
