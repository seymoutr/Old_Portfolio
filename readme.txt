//
//  HospitalData.swift
//  UCStrokeApp_COOP_2017
//
//  Created by Beyette Group Admin on 2/20/17.
//  Copyright © 2017 University of Cincinnati. All rights reserved.
//

import Foundation
import UIKit
import CoreLocation

class HospitalData: CLLocationManager, CLLocationManagerDelegate{
    
    var latitude = 0.0
    var longitude = 0.0
    
    //function that takes the URL of our text file and gets the name, designation, and placeid. It will save this to a hashtable based off placeid and hospital object. The hospital object will have "" and 0.0 as placeholders for later information retrieval. The function then return this hashtable.
    func loadMyJSON(filePath: URL)-> [String: Hospital] {
        do{
            let data = try Data(contentsOf: filePath)
            let allListing = try JSONSerialization.jsonObject(with: data) as![String: AnyObject]
            let all = allListing["list"] as! [AnyObject]
            
            var hashtable: [String: Hospital] = [:]
            let hospitalDistVal = 0.0
            let hospitalDistStrg = ""
            let hospitalDurationVal = 0.0
            let hospitalDurationStrg = ""
            let address = ""
            var name = ""
            var place_id = ""
            var designation = ""
            
            for index in 0...all.count-1 {
                let aObject = all[index] as! [String: AnyObject]
                
                if aObject["name"] == nil{
                    name = "N/A"
                }else{
                    name = aObject["name"] as! String
                }
                
                if aObject["place_id"] == nil{
                    place_id = "N/A"
                }else{
                    place_id = aObject["place_id"] as! String
                }
                
                if aObject["designation"] == nil{
                    designation = "N/A"
                }else{
                    designation = aObject["designation"] as! String
                }
                
                let hospitalItem = Hospital(fromName: name, hospitalPlace_id: place_id, hospitalDistance: hospitalDistStrg, hospitalDistanceValue: hospitalDistVal, hospitalDuration: hospitalDurationStrg, hospitalDurationValue: hospitalDurationVal, hospitalDesignation: designation, hospitalAddress: address)
                
                hashtable[hospitalItem.place_id] = hospitalItem
            }
            return hashtable
        }catch{
            print("Could not load JSON")
            return [:]
        }
    }
    
    
    //function accepts a hashtable paired with a hospital object (myList) which is from the saved txt file. This will do a nearby search for nearest hospitals with keyword: emergency. Taking the results and comparing to the list we have it will save the matching results to new hashtable and return it.
    func getJSON(myList: [String: Hospital])-> [String: Hospital]{
        getMyDist()
        let url = URL(string:"https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=\(latitude),\(longitude)&radius=20000&types=hospital&keyword=emergency&key=AIzaSyC6SaPks1S2NyWy9Q8fSZrZlQCBB9XGKDo")
        var count = 0
        do{
            let allNamesData = try Data(contentsOf: url!)
            let allNames = try JSONSerialization.jsonObject(with: allNamesData) as! [String: AnyObject]
        
            var hashtable: [String: Hospital] = [:]
            if  let arrayJSON = allNames["results"] as? [AnyObject]{
                for i in 0...arrayJSON.count-1{
                    let JSONobject = arrayJSON[i] as! [String: AnyObject]
                    let placeid = JSONobject["place_id"]
                    let name = JSONobject["name"]
                    let address = JSONobject["vicinity"]
                    
                    //checks myList[index] place_id with searched placed_id and if they are the same it creates the hospital object with place_id, name, address, designation. all other hospital values are "" or 0.0.
                    if (myList.index(forKey: placeid as! String) != nil) == true{
                        let hospitalObject = Hospital(fromName: name as! String, hospitalPlace_id: placeid as! String , hospitalDistance: "", hospitalDistanceValue: 0.0 ,hospitalDuration: "", hospitalDurationValue: 0.0, hospitalDesignation: (myList[placeid as! String]?.designation)!,hospitalAddress: address as! String)
                        hashtable[hospitalObject.place_id] = hospitalObject
                    }else{
                        count = count+1
                        print("HospitalObject #\(count) out of 20 not apart of list")
                    }
                }
                return hashtable
            }
            return [:]
        }catch{
            print("Could not get a search for hospitals.")
            return [:]
        }
    }
    
    //function takes the placeid of a given object, sends it to googles distance matrix api and gets duration and distance from your current location to given place.
    func getDetailsJSON(placeid: String)->[String: AnyObject]{
        getMyDist()
                
        let distMatrixAPI = "https://maps.googleapis.com/maps/api/distancematrix/json?units=imperial&origins=\(latitude),\(longitude)&departure_time=now&destinations=place_id:" + placeid + "&sensor=false&key=AIzaSyC6SaPks1S2NyWy9Q8fSZrZlQCBB9XGKDo"
        let url = URL(string: distMatrixAPI)
        do {
            let allData = try Data(contentsOf: url!)
            let allDistMatrix = try JSONSerialization.jsonObject(with: allData) as! [String: AnyObject]
            if  let rows = allDistMatrix["rows"]! as? [AnyObject] {
                if let arrayJSON = rows[0] as? [String: AnyObject] {
                    return arrayJSON
                }
            }
            print("Could not get JSON")
            return [:]
        } catch {
            print("Could not get Distance matrix API to work")
            return [:]
        }
    }
    
    //function that gets your current location and updates it if needed
    func getMyDist(){
        let locationManager = CLLocationManager()
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.requestWhenInUseAuthorization()
        locationManager.startUpdatingLocation()
        
        latitude = locationManager.location!.coordinate.latitude //40.7128
            
        longitude = locationManager.location!.coordinate.longitude //-74.0059
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        //add code if we update location
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("Error while getting location" + error.localizedDescription)
    }
    
}


//class for hospital objects, given a hash value of its placeid
public class Hospital: NSObject{
    var name: String
    var place_id: String
    var distance: String
    var distanceValue: Double
    var duration: String
    var durationValue: Double
    var designation: String
    var address: String
    
    override public var hashValue: Int {
        return self.place_id.hash
    }
    
    init(fromName hospitalName: String, hospitalPlace_id: String , hospitalDistance: String, hospitalDistanceValue: Double ,hospitalDuration: String, hospitalDurationValue: Double, hospitalDesignation: String, hospitalAddress: String) {
        
        self.name = hospitalName
        self.place_id = hospitalPlace_id
        self.distance = hospitalDistance
        self.distanceValue = hospitalDistanceValue
        self.duration = hospitalDuration
        self.durationValue = hospitalDurationValue
        self.designation = hospitalDesignation
        self.address = hospitalAddress
    }
}

//
//  FirstTimeViewScreenController.swift
//  UCStrokeApp_COOP_2017
//
//  Created by Beyette Group Admin on 2/21/17.
//  Copyright © 2017 University of Cincinnati. All rights reserved.
//

import Foundation
import UIKit
import CoreLocation


class FirstTimeViewScreenController: UIViewController, CLLocationManagerDelegate{
    @IBOutlet weak var firstTimeButton: UIButton!
    
    //loads in myDefaults settings for this app
    let myDefaults = UserDefaults.standard
    let key1 = "randomization"
    let key2 = "disclaimer"
    let key3 = "calling"
    let key4 = "firstTimeBool"
    let key5 = "reset"
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        //get the true/false of key2, key4, and key5 settings value
        let disclaimerSet = myDefaults.object(forKey: key2) as? Bool
        let firstTimeSet = myDefaults.object(forKey: key4) as? Bool
        let resetSet = myDefaults.object(forKey: key5) as? Bool
        
        if (disclaimerSet == false && firstTimeSet == true && resetSet == true){
                myDefaults.set(false, forKey: key5) // set reset to false to ensure app doesnt reset outside of resetButton location
            self.performSegue(withIdentifier: "toFast", sender: nil) // goes to FastViewController
        }
        firstTimeButton.layer.cornerRadius = 6 // curves the edges of firstTimeButton
    }
    
    override func viewDidAppear(_ animated: Bool) {
        //shows disclaimer if it is enabled in settings, this is done after view appeared to make sure settings are loaded in
        if let disclaimerNum = myDefaults.object(forKey: key2) {
            if Int(disclaimerNum as! NSNumber) == 1 {
                showDisclaimer()
            }
        }
    }
    
    override func viewWillAppear(_ animated: Bool) {
        //hides the navigation bar since its not needed on this page
        self.navigationController?.isNavigationBarHidden = true
    }
    
    //Function that show disclaimer popup
    func showDisclaimer(){
        let disclaimer = UIAlertController(title: "Disclaimer here", message: "message", preferredStyle: .alert)
        let cancelAction = UIAlertAction(title: "ok", style: .cancel, handler: nil)
        disclaimer.addAction(cancelAction)
        present(disclaimer, animated: true, completion: nil)
    }
    
}
//
//  FASTViewController.swift
//  UCStrokeApp_COOP_2017
//
//  Created by Beyette Group Admin on 2/17/17.
//  Copyright © 2017 University of Cincinnati. All rights reserved.
//

import Foundation
import UIKit
import CoreLocation


class FASTViewController: UIViewController, UIGestureRecognizerDelegate{
    
    @IBOutlet weak var timeQuestionLabel: UILabel!
    @IBOutlet var timePicker: UIDatePicker!
    @IBOutlet var atWhatTime: UILabel!
    @IBOutlet var elapsedTimeLabel: UILabel!
    @IBOutlet var timeSegVar: UISegmentedControl!
    var secondsElapsed: Int = 0 //variable contains the elapsed time from the time the patient had the stroke to the current time.
    
    // segmented controls are used for selecting answers to questions, .selectedSegmentIndex gives -1 by default. If "Yes" is selected gives 0. If "No" is selected gives 1.
    @IBOutlet var faceDroop: UISegmentedControl!
    @IBOutlet var armDrift: UISegmentedControl!
    @IBOutlet var abnormalSpeech: UISegmentedControl!
    @IBOutlet weak var nextButtonFAST: UIButton!
    
    //load in default for reset button
    let myDefaults = UserDefaults.standard
    let key5 = "reset" //reset bool
    
    //create locationManager to access CLLocation functions
    //create cellData which is used for NotaStrokeViewController data
    let locationManager = CLLocationManager()
    var cellData = [Hospital]()
    
    
    override func viewDidAppear(_ animated: Bool) {
        locationManager.requestWhenInUseAuthorization() //asks user to allow app to use their location within the app
        myPatient.fastScore  = 0
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.navigationItem.setHidesBackButton(true, animated: false)
        self.navigationController?.isNavigationBarHidden = false //unhides the navigation bar hidden from previous view
        myDefaults.set(true, forKey: key5)
        
        // hides the items below upon loading
        nextButtonFAST.isHidden = true
        timeQuestionLabel.isHidden = true
        timeSegVar.isHidden = true
        elapsedTimeLabel.isHidden = true
        timePicker.isHidden = true
        atWhatTime.isHidden = true
        nextButtonFAST.isHidden = true
    }
    
    // function assesses if FAST is positive given user information and returns True:False
    func isStroke() -> Bool {
        if (faceDroop.selectedSegmentIndex == 0 || armDrift.selectedSegmentIndex == 0 || abnormalSpeech.selectedSegmentIndex == 0){
            return true
        }
        timePicker.isHidden = true
        timeSegVar.selectedSegmentIndex = -1
        return false
    }
    
    // function checks to see if all questions have been answered or not and returns True:False
    func allAnswered() -> Bool {
        if (faceDroop.selectedSegmentIndex == -1 || armDrift.selectedSegmentIndex == -1 || abnormalSpeech.selectedSegmentIndex == -1){
            return false
        }
        return true
    }
    
    // function puts correct items shown hidden or not hidden
    @IBAction func FASTsegAction(_ sender: UISegmentedControl) {
        if (allAnswered() && isStroke()) {
            timeQuestionLabel.isHidden = false
            timeSegVar.isHidden = false
        }
        else if (allAnswered()) {
            nextButtonFAST.isHidden = false
            timeQuestionLabel.isHidden = true
            timeSegVar.isHidden = true
            elapsedTimeLabel.isHidden = true
            atWhatTime.isHidden = true
        }
    }
    
    // Function on the "Next" button method.
    // If all the questions are answered and there is a stroke, then segue "Elapsed Time" to the TimeViewController.
    // If all the questions are answered and there is not a stroke, then segue "Not a Stroke" to the NotaStrokeViewController
    @IBAction func Next(_ sender: UIButton) {
        if timeSegVar.selectedSegmentIndex == 0{ // today
            lastKnownNormal = timePicker.date
        }else if timeSegVar.selectedSegmentIndex == 1{ // yesterday
            let yesterday = Calendar.current.date(byAdding: .day, value: -1, to: timePicker.date)
            lastKnownNormal = yesterday!
        }else if timeSegVar.selectedSegmentIndex == 2{ // more than two days ago
            let twoDays = Calendar.current.date(byAdding: .day, value: -2, to: timePicker.date)
            lastKnownNormal = twoDays!
        }
        // all scoring of myPatient and myQuestion is done here before segue
        if faceDroop.selectedSegmentIndex == 0{
            myPatient.fastScore += 1
            myQuestion.faceDroop = "Yes"
        }else{
            
            myQuestion.faceDroop = "No"
        }
        if armDrift.selectedSegmentIndex == 0 {
            myPatient.fastScore += 1
            myQuestion.armDrift = "Yes"
        }else{
            
            myQuestion.armDrift = "No"
        }
        if abnormalSpeech.selectedSegmentIndex == 0{
            myPatient.fastScore += 1
            myQuestion.abnSpeech = "Yes"
        }else{
            
            myQuestion.abnSpeech = "No"
        }
        //checks to see if it is a stroke(secondsElapsed)or not a stroke and which segue to go too based off this logic
        if (allAnswered() == true && isStroke() == true && secondsElapsed <= 21600) {
            self.performSegue(withIdentifier: "toCSTAT", sender: self)
        }
        else if (allAnswered() == true && isStroke() == false) {
            self.performSegue(withIdentifier: "Not a Stroke", sender: self)
        }
            
        else if (allAnswered() == true && isStroke() == true && secondsElapsed > 21600 ){
            self.performSegue(withIdentifier: "toPSC", sender: self)
        }
        else {
            print("Finish questions")
        }
    }
    
    // This is for iPhone users only
    // Will send a buttonName value to the OptionsViewContoller
    // this value will determine the back button's text displayed in that ViewController
    var buttonName = 1
    
    @IBAction func optionsButton(_ sender: UIButton) {
        self.performSegue(withIdentifier: "FAST", sender: self) // performs the segue to OptionsViewController on iPhone
    }
    
    
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if segue.identifier == "FAST" {
            if let DestViewController = segue.destination as? OptionsViewController {
                DestViewController.buttonName = self.buttonName // transfers buttonName value
            }
        }
        if segue.identifier == "Not a Stroke" { //this is for both iphone and ipad
            if let DestViewController = segue.destination as? NotaStrokeViewController {
                DestViewController.cellData = self.cellData // tranfers cellData list
            }
        }
    }
    // long press functions here on Labels of questions on FASTViewController, will go to local HTML file. These methods are called when the user holds his/her finger on each Label in this view.
    @IBAction func longPressFace(_ gestureRecognizer: UILongPressGestureRecognizer) {
        
        if gestureRecognizer.state == .began {
            print("Long Press Face")
            self.performSegue(withIdentifier: "droopHelp", sender: self)
        }
    }
    @IBAction func longPressArm(_ gestureRecognizer: UILongPressGestureRecognizer) {
        if gestureRecognizer.state == .began {
            print("Long Press Arm")
            self.performSegue(withIdentifier: "armHelp", sender: self)
        }
    }
    @IBAction func longPressSpeech(_ gestureRecognizer: UILongPressGestureRecognizer) {
        if gestureRecognizer.state == .began {
            print("Long Press Speech")
            self.performSegue(withIdentifier: "speechHelp", sender: self)
        }
    }
    
    //MARK: Time Methods
    
    // Time segment item in ViewContoller when selected
    //NOTE: all times in the function will be in based off device time, but when accsessing this time outside this function(like in a print() statement) will be in UTC time
    @IBAction func TimeSeg(_ sender: UISegmentedControl) {
        
        timePicker.setDate(NSDate() as Date, animated: true) //sets the date
        timePicker.maximumDate = nil //sets max date
        elapsedTimeLabel.isHidden = true
        
        if sender.selectedSegmentIndex == 0 || sender.selectedSegmentIndex == 1  { //segment 0 is today, segment 1 is yesterday
            
            timePicker.isHidden = false
            atWhatTime.isHidden = false
            
            nextButtonFAST.isHidden = true
            
        }else{ // more than two days automatically sets time to 48 hours
            
            timePicker.isHidden = true
            atWhatTime.isHidden = true
            let hours = 48
            let minutes = 0
            elapsedTimeLabel.isHidden = false
            elapsedTimeLabel.text = "Elapsed time: \(hours) hours \(minutes) minutes \n"
            secondsElapsed = 172800
            nextButtonFAST.isHidden = false
        }
        
    }
    
    //Time segment item in ViewContoller when activated(moved)
    @IBAction func TimePickerAction(_ sender: UIDatePicker) {
        
        elapsedTimeLabel.isHidden = false
        let strokeDate = timePicker.date //gets the time at which the stroke occured
        secondsElapsed = Int(-strokeDate.timeIntervalSinceNow) //gets the number of seconds between now and the time of stroke
        //if the stroke occured yestereday adds 24 hours to interval
        if (timeSegVar.selectedSegmentIndex == 1){
            secondsElapsed += 24*3600
        }
        else {
            timePicker.maximumDate = NSDate() as Date
        }
        
        //calculates the hours and minutes since the stroke
        let hours = Int(secondsElapsed/3600)
        let minutes = Int((secondsElapsed - hours*3600)/60)
        elapsedTimeLabel.isHidden = false //sets and unhides the red lable at bottom of screen
        
        if hours <= 0 && minutes <= 0 {
            elapsedTimeLabel.text = ""
            nextButtonFAST.isHidden = true
        }else{
            elapsedTimeLabel.text = "Elapsed time: \(hours) hours \(minutes) minutes \n"
            print(timePicker.date)
            if secondsElapsed <= 21600 { //checks to see if time selected is < 6 hours to determine to goto CSTAT(PSCViewController) or next(more questions)
                nextButtonFAST.isHidden = false
            }else{
                nextButtonFAST.isHidden = false
            }
        }
    }
}

//
//  CSTATAgeMonthBlinkGripViewController.swift
//  UCStrokeApp_COOP_2017
//
//  Created by Beyette Group Admin on 2/20/17.
//  Copyright © 2017 University of Cincinnati. All rights reserved.
//

import Foundation
import UIKit

class CSTATAgeMonthBlinkGripViewController: UIViewController{
    
    @IBOutlet weak var eyeSeg: UISegmentedControl!
    @IBOutlet weak var armSeg: UISegmentedControl!
    @IBOutlet weak var ageSeg: UISegmentedControl!
    @IBOutlet weak var monthSeg: UISegmentedControl!
    @IBOutlet weak var blinkSeg: UISegmentedControl!
    @IBOutlet weak var gripSeg: UISegmentedControl!
    
    @IBOutlet weak var nextToPSC: UIButton!
    @IBOutlet weak var nextToCSC: UIButton!
    
    // checks the user defaults for setting values
    // we will be using the randomization if it is turned on
    let myDefaults = UserDefaults.standard
    let key1 = "randomization"
    
    override func viewDidLoad() {
        nextToPSC.isHidden = true
        nextToCSC.isHidden = true
    }
    override func viewDidAppear(_ animated: Bool) {
        myPatient.cstatScore = 0
        checkAnswers()
    }
    
    //Function that on segmented action checks to see if all questions are answered
    @IBAction func SegAction(_ sender: UISegmentedControl) {
        checkAnswers()
    }
    
    // Function performs check of answers
    func checkAnswers() {
        let eye = eyeSeg.selectedSegmentIndex
        let arm = armSeg.selectedSegmentIndex
        let age = ageSeg.selectedSegmentIndex
        let month = monthSeg.selectedSegmentIndex
        let blink = blinkSeg.selectedSegmentIndex
        let grip = gripSeg.selectedSegmentIndex
        
        //guards against when the questions aren't completed
        guard eye == -1 || arm == -1 || age == -1 || month == -1 || blink == -1 || grip == -1 else {
            if eye == 0 || arm == 0 || age ==  0 || month == 0 || blink == 0 || grip == 0   {
                // if any CSTAT question is yes...
                checkrandom() // if CSTATpositive is false, then execute checkrandom() function
            }else{
                nextToPSC.isHidden = false
                nextToCSC.isHidden = true
            }
            return
        }
    }
    
    // performs randomization feature if set in the settings view
    func checkrandom() {
        let eye = eyeSeg.selectedSegmentIndex
        let arm = armSeg.selectedSegmentIndex
        let age = ageSeg.selectedSegmentIndex
        let month = monthSeg.selectedSegmentIndex
        let blink = blinkSeg.selectedSegmentIndex
        let grip = gripSeg.selectedSegmentIndex
        if let randomization = myDefaults.object(forKey: key1) {
            if Int(randomization as! NSNumber) == 1 {
                // make random number that will be either 0 or 1
                let randnum = arc4random_uniform(2)
                if randnum == 0 {
                    // stroke suspected, Go to CSC
                    nextToPSC.isHidden = true
                    nextToCSC.isHidden = false
                }else{
                    // stroke suspected, Go to PSC
                    nextToPSC.isHidden = false
                    nextToCSC.isHidden = true
                }
            }
            else if eye == 0 || arm == 0 || age ==  0 || month == 0 || blink == 0 || grip == 0 {
                // severe stroke suspected, Go to CSC
                nextToPSC.isHidden = true
                nextToCSC.isHidden = false
            }else{
                // stroke suspected, Go to PSC
                nextToPSC.isHidden = false
                nextToCSC.isHidden = true
            }
        }
    }
    
    
    
    //The CSTAT segue is for iPhone users only
    //Will send a buttonName value to the OptionsViewContoller
    //this value will determine the Back button name in that ViewController
    
    @IBAction func PlusButton(_ sender: UIButton) {
        self.performSegue(withIdentifier: "CSTAT", sender: self) //performs segue
    }
    var buttonName = 3
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if segue.identifier == "CSTAT" {
            if let DestViewController = segue.destination as? OptionsViewController {
                DestViewController.buttonName = self.buttonName //transfers answers
            }
        }
        
        //For both iphone and ipad here are the question answers and scoring computations.
        if segue.identifier == "toCSC" {
            if eyeSeg.selectedSegmentIndex == 0{
                myQuestion.eyeMove = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.eyeMove = "No"
            }
            if armSeg.selectedSegmentIndex == 0{
                myQuestion.armDroop = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.armDroop = "No"
            }
            if ageSeg.selectedSegmentIndex == 0{
                myQuestion.age = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.age = "No"
            }
            if monthSeg.selectedSegmentIndex == 0{
                myQuestion.month = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.month = "No"
            }
            if blinkSeg.selectedSegmentIndex == 0{
                myQuestion.eyeCommand = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.eyeCommand = "No"
            }
            if gripSeg.selectedSegmentIndex == 0{
                myQuestion.gripCommand = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.gripCommand = "No"
            }
        }
        if segue.identifier == "toPSC" {
            if eyeSeg.selectedSegmentIndex == 0{
                myQuestion.eyeMove = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.eyeMove = "No"
            }
            if armSeg.selectedSegmentIndex == 0{
                myQuestion.armDroop = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.armDroop = "No"
            }
            if ageSeg.selectedSegmentIndex == 0{
                myQuestion.age = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.age = "No"
            }
            if monthSeg.selectedSegmentIndex == 0{
                myQuestion.month = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.month = "No"
            }
            if blinkSeg.selectedSegmentIndex == 0{
                myQuestion.eyeCommand = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.eyeCommand = "No"
            }
            if gripSeg.selectedSegmentIndex == 0{
                myQuestion.gripCommand = "Yes"
                myPatient.cstatScore += 1
            }else{
                myQuestion.gripCommand = "No"
            }
        }
    }
}







